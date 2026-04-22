# LibSQL + Drizzle ORM Setup Guide

Continue this after [express-mvc.md](./express-mvc.md).

This guide upgrades your MVC app from in-memory arrays to a real database using:

1. LibSQL as the database driver.
2. Drizzle ORM for schema and queries.

## 1) Why add Drizzle after MVC

In [express-mvc.md](./express-mvc.md), your model used arrays.
That is good for learning structure, but data resets every time the server restarts.

With LibSQL + Drizzle:

1. Data is persistent in a database file.
2. Your model layer becomes production-style data access code.
3. You can define schema in code and keep it versioned.

## 2) Install dependencies

From your project root (where your `package.json` is), run:

```powershell
npm install drizzle-orm @libsql/client dotenv
npm install -D drizzle-kit
```

Why these packages:

1. `drizzle-orm`: query builder and ORM.
2. `@libsql/client`: LibSQL client used by Drizzle.
3. `drizzle-kit`: migration and schema tooling.
4. `dotenv`: load `DATABASE_URL` from `.env`.

## 3) Set environment variables

Update `.env` in project root:

```env
PORT=3000
DATABASE_URL=file:./local.db
```

## 4) Add Studio script to package.json

Run:

```powershell
npm pkg set scripts.studio="drizzle-kit studio"
```

Expected new scripts:

```json
{
  "scripts": {
		"studio": "drizzle-kit studio"
  }
}
```

## 5) Create database folders

Create all database files first so students can see the full picture early.

Use this structure in your MVC project:

```text
project-root/
├─ src/
│  ├─ db/
│  │  ├─ connection.js
│  │  ├─ schema.js
│  │  └─ seed.js
│  ├─ routes/
│  ├─ controllers/
│  └─ models/
├─ drizzle.config.js
├─ .env
└─ index.js
```

Create files:

```powershell
mkdir src\db
ni src\db\schema.js
ni src\db\connection.js
ni src\db\seed.js
ni drizzle.config.js
```

## 6) Define your schema

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
	email: text('email').notNull().unique()
});
```

## 7) Configure Drizzle

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

## 8) Create DB connection

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
	url: process.env.DATABASE_URL || 'file:./local.db',
});

// Wraps the client with Drizzle so models can run typed queries
export const db = drizzle(client);
```

## 9) Configure seed command and seed script

Set a script so students can reset and reseed quickly.

Run:

```powershell
npm pkg set scripts.db="node src/db/seed.js"
```

Then add this content:

### src/db/seed.js

```js
/* ----------------------------
   SEED + RESET SCRIPT
------------------------------ */
import fs from 'fs';
import path from 'path';
import { execSync } from 'child_process';
import { fileURLToPath } from 'url';
import 'dotenv/config';

import { users } from './schema.js';

const sampleUsers = [
	{ name: 'Alice', email: 'alice@example.com' },
	{ name: 'Bob', email: 'bob@example.com' },
	{ name: 'Charlie', email: 'charlie@example.com' }
];

// Inserts starter users into the users table
export async function seed(db) {
	await db.insert(users).values(sampleUsers);
	console.log(`Inserted ${sampleUsers.length} users`);
}

const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);
const projectRoot = path.resolve(__dirname, '..', '..');

const dbUrl = process.env.DATABASE_URL || 'file:./local.db';
const dbPath = dbUrl.replace('file:', '');
const absoluteDbPath = path.resolve(projectRoot, dbPath);

async function resetDatabase() {
	try {
		// Step 1: delete old database file if it exists.
		if (fs.existsSync(absoluteDbPath)) {
			fs.unlinkSync(absoluteDbPath);
			console.log(`Deleted old database: ${dbPath}`);
		}

		// Step 2: recreate tables from schema.
		console.log('Creating tables from schema...');
		execSync('npx drizzle-kit push', {
			cwd: projectRoot,
			stdio: 'inherit'
		});

		// Step 3: insert sample users.
		console.log('Seeding database...');
		const { db } = await import('./connection.js');
		await seed(db);

		console.log('Done! Database is ready.');
	} catch (error) {
		console.error('Failed to reset database:', error.message);
		process.exit(1);
	}
}

// Runs the full reset + seed process
resetDatabase();
```

## 10) Initialize and seed the database

Recommended first run:

```powershell
npm run db
```

Expected result:

1. `local.db` is created (or recreated) in project root.
2. `users` table is created from schema.
3. Sample users are inserted.

Use `npm run db` after schema changes in this teaching project, so students always work with a clean and known dataset.

## 11) Use Studio to verify database updates

Drizzle Studio is a browser-based table viewer/editor for your database.

Think of it as a GUI for checking whether your API actions really changed data.

Official guide:

- https://orm.drizzle.team/drizzle-studio/overview

After running `npm run db`, open Drizzle Studio to inspect your actual database rows.

Run:

```powershell
npm run studio
```

How to verify:

1. Open the `users` table.
2. Confirm sample rows exist (`Alice`, `Bob`, `Charlie`).
3. Test one API call (for example `POST /users` in Postman).
4. Refresh Studio and confirm the new row appears.
5. Test one `PUT /users/:id` and confirm row values changed.
6. Test one `DELETE /users/:id` and confirm row is removed.

What this teaches:

1. API calls are really changing database state.
2. Controller and model code is wired correctly.
3. Troubleshooting is easier when you can see table data directly.

When to use Studio in class:

1. After `npm run db` to confirm seed data exists.
2. After each CRUD test to verify expected row changes.
3. During debugging when API response and DB state do not match.

## 12) Replace array model with Drizzle model

Update your MVC model to query the database.

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
		.set(payload)
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


## 13) Make controllers async

Because database calls are async, update your controller functions to use `async/await`.

Example pattern:

```js
/* ----------------------------
   IMPORTS
------------------------------ */
import {
	selectUsers,
	selectUserById,
	insertUser,
	updateUser,
	deleteUser as deleteUserModel
} from '../models/userModel.js';

/* ----------------------------
   READ HANDLERS
------------------------------ */

// GET /users
export const getUsers = async (req, res) => {
	const result = await selectUsers();

	res.status(200).json(result);
	return;
};

// GET /users/:id
export const getUserById = async (req, res) => {
	const payload = {
		id: req.params.id
	};

	const result = await selectUserById(payload);
	if (!result) {
		res.status(404).json({ message: 'User not found' });
		// return stops function execution to prevent sending a second response later
		return;
	}

	res.status(200).json(result);
	return;
};

/* ----------------------------
   CREATE HANDLER
------------------------------ */

// POST /users
export const postUser = async (req, res) => {
	const { name, email } = req.body;

	if (!name || !email) {
		res.status(400).json({ message: 'name and email are required' });
		// return stops function execution to prevent sending a second response later
		return;
	}

	const payload = {
		name,
		email
	};

	const result = await insertUser(payload);
	res.status(201).json(result);
	return;
};

/* ----------------------------
   UPDATE HANDLER
------------------------------ */

// PUT /users/:id
export const putUserById = async (req, res) => {
	const payload = {
		...req.body,
		id: req.params.id
	};

	const result = await updateUser(payload);
	if (!result) {
		res.status(404).json({ message: 'User not found' });
		// return stops function execution to prevent sending a second response later
		return;
	}

	res.status(200).json(result);
	return;
};

/* ----------------------------
   DELETE HANDLER
------------------------------ */

// DELETE /users/:id
export const deleteUserById = async (req, res) => {
	const payload = {
		id: req.params.id
	};

	const result = await deleteUserModel(payload);
	if (!result) {
		res.status(404).json({ message: 'User not found' });
		// return stops function execution to prevent sending a second response later
		return;
	}

	res.status(200).json({ message: 'User deleted' });
	return;
};
```

Use this as the template for your full `userController.js` async handlers.

## 14) Quick test flow

1. Run reset + seed once: `npm run db`.
2. Start server: `npm run dev`.
3. Test in Postman:
   - `POST /users`
   - `GET /users`
   - `PUT /users/:id`
   - `DELETE /users/:id`

If server restarts and data remains, your database setup is working.

## 15) Common troubleshooting

1. `DATABASE_URL` missing:
   - Ensure `.env` has `DATABASE_URL=file:./local.db`.
   - Ensure `dotenv/config` is imported in DB config files.
2. Table not found:
   - Run `npm run db` to recreate database and reseed.
3. `Cannot use import statement outside a module`:
   - Ensure `"type": "module"` exists in `package.json`.
4. Permission issues on school/staff machine:
   - Use a project folder where you have write access.
   - Avoid protected system folders.

## 16) What to teach students explicitly

1. MVC did not change. Only the model data source changed.
2. Keep controllers focused on HTTP and validation.
3. Keep database logic inside model files.
4. In this module, run `npm run db` after schema updates.
5. Use `npm run db` when you want a clean reset with sample users.
6. Students must understand generated or AI-suggested code before submission.

