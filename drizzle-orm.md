# Drizzle ORM Detailed Guide

Continue this after [libsql-setup.md](./libsql-setup.md).

This guide explains Drizzle ORM in practical depth:

1. What Drizzle ORM is and why people use it
2. How schema, migration, and query layers connect
3. How to write safe CRUD and join queries
4. How to write maintainable query code and avoid common mistakes

## 1) What is Drizzle ORM

Drizzle ORM is a lightweight TypeScript-first ORM and query builder.

For beginner backend learning, the key idea is:

1. You define your database structure in code (schema).
2. You run migration/push tools to apply schema to your database.
3. You query through Drizzle APIs instead of raw SQL strings in most places.

Why this is useful:

1. Better readability and maintainability than scattered SQL strings.
2. Safer queries with fewer typo-style bugs.
3. Clear schema source of truth in your project.

## 2) Drizzle architecture in one picture

Think in three layers:

1. Schema layer
	- File example: `src/db/schema.js`
	- Defines tables, columns, and constraints.
2. Connection layer
	- File example: `src/db/connection.js`
	- Creates database client and exports `db`.
3. Data access layer (models)
	- File example: `src/models/userModel.js`
	- Uses `db.select/insert/update/delete` to run queries.

## 3) Core packages and their roles

Common packages used with SQLite/LibSQL setup:

1. `drizzle-orm`
	- Runtime ORM/query APIs.
2. `drizzle-kit`
	- Schema tooling for push/migration/studio.
3. `@libsql/client`
	- LibSQL connection client used by Drizzle.

Install command:

```powershell
npm install drizzle-orm @libsql/client dotenv
npm install -D drizzle-kit
```

## 4) Basic project structure

A typical structure for this learning repo style:

```text
project-root/
├─ src/
│  ├─ db/
│  │  ├─ connection.js
│  │  ├─ schema.js
│  │  └─ seed.js
│  ├─ models/
│  ├─ controllers/
│  └─ routes/
├─ drizzle.config.js
├─ .env
└─ index.js
```

## 5) Environment and config

### .env

```env
DATABASE_URL=file:./local.db
```

### drizzle.config.js

```js
/* ----------------------------
   DRIZZLE CONFIG
------------------------------ */
import 'dotenv/config';

// Tells drizzle-kit where schema is and which DB to connect to
export default {
	schema: './src/db/schema.js',
	dialect: 'sqlite',
	dbCredentials: {
		url: process.env.DATABASE_URL || 'file:./local.db'
	}
};
```

How this example works:

1. `import 'dotenv/config'` loads environment variables before this file runs.
2. `schema` tells Drizzle Kit where your table definitions live.
3. `dialect: 'sqlite'` tells Drizzle which SQL syntax family to use.
4. `dbCredentials.url` gives Drizzle the database location.
5. `process.env.DATABASE_URL || 'file:./local.db'` gives you a safe fallback if `.env` is missing.

This file is not your app database connection. It is the configuration Drizzle Kit uses to understand your database structure and generate or apply schema changes.

## 6) Connection setup

### src/db/connection.js

```js
/* ----------------------------
   DATABASE CONNECTION
------------------------------ */
import 'dotenv/config';
import { createClient } from '@libsql/client';
import { drizzle } from 'drizzle-orm/libsql';

// Creates the low-level LibSQL client
const client = createClient({
	url: process.env.DATABASE_URL || 'file:./local.db'
});

// Wraps the client with Drizzle so models can run typed queries
export const db = drizzle(client);
```

How this example works:

1. `createClient(...)` creates the low-level connection to LibSQL.
2. `drizzle(client)` wraps that connection with Drizzle ORM features.
3. `export const db` gives the rest of your app one shared database object.
4. Controllers and models use `db` to build queries without writing raw SQL everywhere.

This file is the runtime connection layer. It is separate from `drizzle.config.js` because one file configures tooling, while this file powers actual database queries in your app.

## 7) Schema definition fundamentals

### src/db/schema.js

```js
/* ----------------------------
   USERS TABLE SCHEMA
------------------------------ */
import { sqliteTable, integer, text } from 'drizzle-orm/sqlite-core';

// Defines the users table structure in SQLite
export const users = sqliteTable('users', {
	id: integer('id').primaryKey({ autoIncrement: true }),
	name: text('name').notNull(),
	email: text('email').notNull().unique(),
	createdAt: integer('created_at', { mode: 'timestamp' }).$defaultFn(() => new Date())
});

export const posts = sqliteTable('posts', {
	id: integer('id').primaryKey({ autoIncrement: true }),
	title: text('title').notNull(),
	content: text('content'),
	authorId: integer('author_id').notNull()
});
```

How this example works:

1. `sqliteTable('users', ...)` defines a table named `users`.
2. The object inside the table call maps column names to column rules.
3. `integer('id')` creates an integer column named `id` in the database.
4. `.primaryKey({ autoIncrement: true })` makes `id` the unique row identifier.
5. `text('name').notNull()` means every row must have a name.
6. `text('email').notNull().unique()` means email is required and cannot repeat.
7. `createdAt` stores a timestamp, and `$defaultFn(() => new Date())` fills it automatically when you insert a row.
8. `posts` is added so later join examples have a matching table.

This schema is the source of truth for the table structure. If the schema says a field is required, the database should enforce that rule.

Important schema concepts:

1. Table name and column names should be explicit and consistent.
2. `notNull()` and `unique()` encode data rules at database level.
3. Primary key and constraints reduce invalid data risk.

## 8) Query patterns for CRUD

### src/models/userModel.js

```js
/* ----------------------------
   IMPORTS
------------------------------ */
import { eq } from 'drizzle-orm';
import { db } from '../db/connection.js';
import { users } from '../db/schema.js';

/* ----------------------------
   READ FUNCTIONS
------------------------------ */

export const selectUsers = async () => {
	const result = await db.select().from(users);
	return result;
};

export const selectUserById = async (payload) => {
	const rows = await db
		.select()
		.from(users)
		.where(eq(users.id, Number(payload.id)));

	const result = rows[0] ?? null;
	return result;
};

/* ----------------------------
   CREATE FUNCTION
------------------------------ */

export const insertUser = async (payload) => {
	const rows = await db
		.insert(users)
		.values({
			name: payload.name,
			email: payload.email
		})
		.returning();

	const result = rows[0];
	return result;
};

/* ----------------------------
   UPDATE FUNCTION
------------------------------ */

export const updateUser = async (payload) => {
	const rows = await db
		.update(users)
		.set({
			name: payload.name,
			email: payload.email
		})
		.where(eq(users.id, Number(payload.id)))
		.returning();

	const result = rows[0] ?? null;
	return result;
};

/* ----------------------------
   DELETE FUNCTION
------------------------------ */

export const deleteUser = async (payload) => {
	const rows = await db
		.delete(users)
		.where(eq(users.id, Number(payload.id)))
		.returning();

	const result = rows.length > 0;
	return result;
};
```

How this example works:

1. The model functions receive a `payload` object instead of separate arguments.
2. `selectUsers()` reads every row from the table.
3. `selectUserById(payload)` uses `payload.id` to build the filter condition.
4. `insertUser(payload)` maps payload fields into database columns with `values(...)`.
5. `updateUser(payload)` uses `set(...)` to describe the columns that change.
6. `deleteUser(payload)` removes exactly one row by `id` and then returns whether deletion happened.

This is the model layer version of ORM syntax: query shape first, data mapping second, result handling last.

## 9) Filtering, sorting, and pagination

```js
/* ----------------------------
   IMPORTS
------------------------------ */
import { desc, like } from 'drizzle-orm';
import { db } from '../db/connection.js';
import { users } from '../db/schema.js';

/* ----------------------------
   SEARCH FUNCTION
------------------------------ */

export const searchUsers = async (payload) => {
	const rows = await db
		.select()
		.from(users)
		.where(like(users.name, `%${payload.keyword || ''}%`))
		.orderBy(desc(users.id))
		.limit(Number(payload.limit) || 10)
		.offset(Number(payload.offset) || 0);

	const result = rows;
	return result;
};
```

How this example works:

1. `select()` says this is a read query.
2. `from(users)` chooses the table.
3. `like(users.name, `%${payload.keyword || ''}%`)` searches for rows whose names contain the keyword.
4. `orderBy(desc(users.id))` sorts newest rows first.
5. `limit(...)` controls how many rows come back.
6. `offset(...)` skips rows for pagination.

This example teaches how query clauses stack together. The chain reads like a sentence: select from users, filter by name, sort by id, then paginate.

### Example data and results

Assume your `users` table currently has this data:

| id | name    | email                |
|---:|---------|----------------------|
| 1  | Alice   | alice@example.com    |
| 2  | Bob     | bob@example.com      |
| 3  | Charlie | charlie@example.com  |
| 4  | Alicia  | alicia@example.com   |
| 5  | Albert  | albert@example.com   |

Now run:

```js
await searchUsers({
	keyword: 'ali',
	limit: 2,
	offset: 0
});
```

Step by step behavior:

1. `like(users.name, '%ali%')` matches `Alice` and `Alicia`.
2. `orderBy(desc(users.id))` sorts matching rows by id: `Alicia (4)`, then `Alice (1)`.
3. `limit(2)` keeps both rows because there are only 2 matches.
4. `offset(0)` skips none.

Expected result:

```json
[
	{ "id": 4, "name": "Alicia", "email": "alicia@example.com" },
	{ "id": 1, "name": "Alice", "email": "alice@example.com" }
]
```

Pagination example (next page):

```js
await searchUsers({
	keyword: 'a',
	limit: 2,
	offset: 2
});
```

How to visualize this second query:

1. Filter `name` containing `a` -> `Alice (1)`, `Charlie (3)`, `Alicia (4)`, `Albert (5)`.
2. Sort desc by id -> `Albert (5)`, `Alicia (4)`, `Charlie (3)`, `Alice (1)`.
3. Offset 2 -> skip `Albert`, `Alicia`.
4. Limit 2 -> return `Charlie`, `Alice`.

Expected result:

```json
[
	{ "id": 3, "name": "Charlie", "email": "charlie@example.com" },
	{ "id": 1, "name": "Alice", "email": "alice@example.com" }
]
```

### How to read ORM query syntax

When you read Drizzle code, parse it as a sentence from top to bottom.

Example shape:

```js
db.select().from(users).where(eq(users.id, 1));
```

How to read this line:

1. `db` = database object connected to your engine.
2. `select()` = choose read operation.
3. `from(users)` = target table.
4. `where(...)` = filtering rule.
5. `eq(users.id, 1)` = predicate expression (`users.id = 1`).

Think of method chaining as query clauses in order.

## 10) ORM syntax anatomy (CRUD)

This section explains the syntax pattern behind each operation, independent of business logic.

### Read syntax anatomy

```js
db.select().from(users).where(eq(users.id, Number(payload.id)));
```

Syntax parts:

1. `select()` defines projection (all columns here).
2. `from(users)` sets source table.
3. `where(...)` narrows rows.
4. `eq(column, value)` creates a safe comparison expression.

### Insert syntax anatomy

```js
db.insert(users).values({ name: payload.name, email: payload.email }).returning();
```

Syntax parts:

1. `insert(users)` chooses target table.
2. `values({...})` maps object keys to table columns.
3. `returning()` asks DB to return inserted rows.

### Update syntax anatomy

```js
db
	.update(users)
	.set({ name: payload.name, email: payload.email })
	.where(eq(users.id, Number(payload.id)))
	.returning();
```

Syntax parts:

1. `update(users)` selects mutation type.
2. `set({...})` describes changed columns.
3. `where(...)` scopes which rows are updated.
4. `returning()` returns updated row snapshot.

### Delete syntax anatomy

```js
db.delete(users).where(eq(users.id, Number(payload.id))).returning();
```

Syntax parts:

1. `delete(users)` chooses delete operation.
2. `where(...)` is critical safety guard.
3. `returning()` allows post-delete confirmation.

### Payload to column mapping rules

Use these rules when writing model functions:

1. Payload key should match model input intent, not always raw DB naming.
2. Map payload explicitly in `values` or `set`.
3. Convert types at boundary (for example `Number(payload.id)`).
4. Keep all mapping logic in model for predictability.

Notes:

1. `like()` handles partial text matching.
2. `limit` + `offset` supports pagination.
3. Keep filtering logic close to the model layer.

## 11) Joins and queries

### Explicit join

![Inner join diagram](https://www.w3schools.com/sql/img_inner_join.png)

This image shows an inner join result. Only rows that match in both tables appear in the output.

```js
/* ----------------------------
   IMPORTS
------------------------------ */
import { eq } from 'drizzle-orm';
import { db } from '../db/connection.js';
import { users, posts } from '../db/schema.js';

/* ----------------------------
   JOIN QUERY
------------------------------ */

export const selectPostsWithAuthor = async () => {
	const result = await db
		.select({
			postId: posts.id,
			title: posts.title,
			authorName: users.name,
			authorEmail: users.email
		})
		.from(posts)
		.innerJoin(users, eq(posts.authorId, users.id));

	return result;
};
```

How this example works:

1. `.select({ ... })` chooses specific columns instead of returning every field.
2. The keys you create, like `postId` and `authorName`, become the shape of the returned object.
3. `.from(posts)` starts from the posts table because posts are the main rows you want.
4. `.innerJoin(users, eq(posts.authorId, users.id))` matches each post with its owning user.
5. `eq(posts.authorId, users.id)` is the actual join condition.

Example data and result:

Assume these rows exist.

`users` table:

| id | name  | email             |
|---:|-------|-------------------|
| 1  | Alice | alice@example.com |
| 2  | Bob   | bob@example.com   |
| 3  | Cindy | cindy@example.com |

`posts` table:

| id | title              | author_id |
|---:|--------------------|----------:|
| 10 | Intro to Node      | 1         |
| 11 | Express Tips       | 2         |
| 12 | Draft Unlinked     | 99        |

When `innerJoin(users, eq(posts.authorId, users.id))` runs:

1. Post `10` matches user `1`.
2. Post `11` matches user `2`.
3. Post `12` has `author_id = 99`, so no user matches.
4. User `3` has no matching post row in this query shape.

Expected result from `selectPostsWithAuthor()`:

```json
[
	{
		"postId": 10,
		"title": "Intro to Node",
		"authorName": "Alice",
		"authorEmail": "alice@example.com"
	},
	{
		"postId": 11,
		"title": "Express Tips",
		"authorName": "Bob",
		"authorEmail": "bob@example.com"
	}
]
```

This is useful when you want a combined response without making separate queries for posts and authors.

## 12) Migrations vs push

Drizzle Kit gives multiple workflows:

1. `drizzle-kit push`
	- Applies schema directly to DB.
	- Great for fast local learning.
2. Migration workflow (`generate` + `migrate`)
	- Produces migration files.
	- Better for team environments and production history.

For beginner class projects, push is often enough.
For team production systems, prefer migration files.

## 13) Error handling patterns

At controller/model boundary:

1. Validate input before model call.
2. Catch database/constraint errors.
3. Return meaningful HTTP status and message.

Example controller-style handling:

```js
/* ----------------------------
   IMPORTS
------------------------------ */
import { insertUser } from '../models/userModel.js';

/* ----------------------------
   CREATE HANDLER
------------------------------ */

export const postUser = async (req, res) => {
	try {
		const payload = {
			name: req.body.name,
			email: req.body.email
		};

		const result = await insertUser(payload);

		res.status(201).json({
			message: 'User created',
			data: result
		});
		return;
	} catch (error) {
		if (String(error.message).includes('UNIQUE')) {
			res.status(409).json({ message: 'Email already exists' });
			return;
		}

		res.status(500).json({ message: 'Internal Server Error' });
		return;
	}
};
```

How this example works:

1. The controller reads the request body and prepares a clean payload object.
2. It checks the required fields before sending data to the model.
3. `insertUser(payload)` keeps database logic out of the controller.
4. Success responds with `201 Created`, which matches a create operation.
5. The `catch` block converts database errors into user-friendly HTTP responses.

This shows how ORM code fits into an Express endpoint: controller for HTTP, model for database work, response for the client.

## 14) Common mistakes and fixes

1. Mistake: table columns and payload keys do not match.
	- Fix: map payload explicitly in model function.
2. Mistake: forgetting `where()` in update/delete.
	- Fix: enforce id-based update/delete helpers and review code.
3. Mistake: doing data logic directly in controller.
	- Fix: keep controller thin, move data logic to model.
4. Mistake: expecting DB to reset automatically.
	- Fix: keep a repeatable seed/reset command.

## 15) Drizzle in MVC responsibility map

1. Route
	- Path and HTTP method mapping only.
2. Controller
	- Reads `req`, validates surface-level input, sends `res`.
3. Model
	- Drizzle schema-level query code.

This separation keeps code easier to test and maintain.

## 16) Practical checklist

Before shipping a Drizzle feature:

1. Schema constraints added (`notNull`, `unique`).
2. Model queries use explicit filters (`eq`, `and`, etc.).
3. Update/delete paths include safe `where` conditions.
4. Create/update return values are controlled and predictable.
5. Seed/reset flow exists for local reproducibility.

## 17) Quick command reference

```powershell
# Apply schema quickly to local DB
npx drizzle-kit push

# Open Drizzle Studio
npx drizzle-kit studio

# Reseed (if script exists in package.json)
npm run db
```

Drizzle ORM helps you keep database code structured, explicit, and production-friendly while still being easy to learn in small MVC projects.

## 18) Can ORM work across different database syntax

Short answer: yes, partially.

Drizzle helps by giving one query API, but databases still have different SQL dialects.

What Drizzle normalizes well:

1. Basic CRUD (`select`, `insert`, `update`, `delete`)
2. Common filters (`eq`, `like`, `and`, `or`)
3. Joins and many common query patterns
4. Schema structure patterns (tables and keys)

What can still differ by database engine:

1. Column types (for example `jsonb`, enum, timestamp behavior)
2. Auto-increment strategy and id generation rules
3. Default value behavior and date/time functions
4. Raw SQL functions and vendor-specific features

In practice, ORM reduces syntax differences, but does not remove them completely.

## 19) Is it easier to migrate to another database

Usually easier than raw SQL projects, but not one-click.

Why ORM helps migration:

1. Query code is centralized in model layer, not scattered SQL strings.
2. Schema is defined in code, so table structure is easier to port.
3. Drizzle supports multiple dialects, so the same code style can be reused.

Why migration can still take work:

1. You may need to adjust column types for the new engine.
2. Existing migrations may need regeneration for target dialect.
3. Some queries using engine-specific SQL must be rewritten.
4. Seed scripts and constraints may behave slightly differently.

Good expectation:

1. Core CRUD usually ports quickly.
2. Advanced features require targeted fixes.

## 20) How to verify portability (practical checklist)

Use this approach to verify whether your Drizzle app is truly portable.

1. Keep model functions database-agnostic.
	- Prefer Drizzle operators over raw SQL.
2. Switch only configuration first.
	- Change `dialect` and DB credentials in `drizzle.config.js`.
3. Recreate schema on the target engine.
	- Run dialect-appropriate migration/push flow.
4. Rerun seed script.
	- Confirm baseline data inserts without constraint/type errors.
5. Retest your API endpoints.
	- Test create, read, update, delete, filtering, and joins.
6. Check edge cases.
	- Unique constraints, null handling, and timestamps.

If all six pass, your ORM layer is effectively portable for that project scope.

## 21) Recommended migration strategy for learners

For teaching projects, migrate in this order:

1. Move schema first.
2. Make seed data work.
3. Make CRUD operations work.
4. Then handle advanced queries and engine-specific features.

This keeps migration risk low and debugging simple.
