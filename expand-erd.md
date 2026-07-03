# Expand the ERD with Plots

Continue this after [design-erd.md](./design-erd.md).

Your task is to expand the ERD by adding a new `Plots` entity. Do not jump to the code yet. Work through each part in order.

## 1) Study the ERD first

Step 1: Open [dbdiagram.io](https://dbdiagram.io/).

Step 2: Read the ERD below carefully.

Step 3: Identify what the new `Plots` table stores.

Step 4: Identify how `Plots` connects to `Users`.

Use this ERD in [dbdiagram.io](https://dbdiagram.io/) as the starting point:

```dbml
Table Users {
	id INT [pk, increment]
	name TEXT [not null]
	email TEXT [not null]
}

Table Plots {
	id INT [pk, increment]
	seed_type INT [not null]
	owner_id INT [not null]
}

Ref: Plots.owner_id > Users.id
```

You can also set the relationship in the following way.
```
Ref: Users.id < Plots.owner_id  
```

## 2) Check the file structure

Step 1: Look at the existing Users MVC files.

Step 2: Notice which Plots files do not exist yet.

Step 3: Create the new Plots files using the same folder pattern.

Use this file structure as your guide:

```text
project-root/
├─ src/
│  ├─ db/
│  │  ├─ connection.js
│  │  ├─ schema.js
│  │  └─ seed.js
│  ├─ routes/
│  │  ├─ userRoutes.js
│  │  └─ plotRoutes.js (new)
│  ├─ controllers/
│  │  ├─ userController.js
│  │  └─ plotController.js (new)
│  └─ models/
│     ├─ userModel.js
│     └─ plotModel.js (new)
├─ drizzle.config.js
├─ .env
└─ index.js
```

## 3) Update the database files

Step 1: Open `src/db/schema.js`.

Step 2: Define both `Users` and `Plots` in the schema.

Step 3: Keep the relationship from `Users.id` to `Plots.owner_id`.

Step 4: Open `src/db/seed.js`.

Step 5: Update it so both users and plots are seeded.

What to do in each database file:

1. `src/db/schema.js`: define the `Users` and `Plots` tables so the database knows the columns, primary keys, and relationship columns.
2. `src/db/seed.js`: insert starter data for both tables so the app has something to work with when you test the new plots feature.
3. Keep the `Users.id` to `Plots.owner_id` relationship in the schema so the plot owner is linked correctly.

## 4) Analyze the existing user seed example

Step 1: Read the current user seed example.

Step 2: Notice what table is imported.

Step 3: Notice how the sample array is structured.

Step 4: Notice how the `seed(db)` function inserts data.

Here is the existing `seed.js` example for users:

```js
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
```

## 5) Extend `seed.js` for plots

Step 1: Analyze the user example above.

Step 2: Add plot seed data using the sample below.

Step 3: Update `seed.js` so it seeds `Plots` too.

Step 4: Keep the user seed and add the plot seed after it.

Analyze this example and update `seed.js` so it seeds `Plots` too. Use this starter plot data:

```js
const samplePlots = [
	{ owner_id: 3, seed_type: 0 },
	{ owner_id: 3, seed_type: 2 },
	{ owner_id: 3, seed_type: 0 },
	{ owner_id: 1, seed_type: 0 },
	{ owner_id: 2, seed_type: 0 },
	{ owner_id: 3, seed_type: 0 },
	{ owner_id: 2, seed_type: 1 },
	{ owner_id: 2, seed_type: 0 },
	{ owner_id: 1, seed_type: 0 },
	{ owner_id: 1, seed_type: 0 }
];
```

## 6) Run the database seed and verify it

Step 1: Run `npm run db` to seed the database.

Step 2: Open Drizzle Studio.

Step 3: Check the tables and confirm the new plot data appears.

After you update the seed file, run `npm run db` to seed the database.

Then open Drizzle Studio and check the tables to view the change.

## 7) Use Users MVC as the template for Plots

Step 1: Open each Users MVC file.

Step 2: Build the matching Plots file beside it.

Step 3: Keep the same structure and CRUD flow.

Step 4: Rename users-specific logic to plots-specific logic.

Finally, use the existing Users MVC files as your guide to make the new Plots MVC files.

Use each Users MVC file as a direct example:

1. `src/routes/userRoutes.js` -> use this as the pattern for `src/routes/plotRoutes.js`.
2. `src/controllers/userController.js` -> use this as the pattern for `src/controllers/plotController.js`.
3. `src/models/userModel.js` -> use this as the pattern for `src/models/plotModel.js`.

## 8) Follow the pattern in each MVC file

### Part A: Routes

Step 1: Open `src/routes/userRoutes.js`.

Step 2: Use it as the pattern for `src/routes/plotRoutes.js`.

Step 3: Change the route file name and controller import to plots.

Step 4: Keep the same CRUD route pattern: list all, get one by id, create one, update one by id, delete one by id.

Step 5: Register endpoints for `/plots` the same way `/users` was registered.

### Part B: Controller

Step 1: Open `src/controllers/userController.js`.

Step 2: Use it as the pattern for `src/controllers/plotController.js`.

Step 3: Keep the same handler structure: one function for GET all, GET by id, POST, PUT by id, and DELETE by id.

Step 4: Change the function names from user-based names to plot-based names.

Step 5: Read `req.params.id` the same way for single-record routes.

Step 6: Read `req.body` and validate the fields that a plot needs before calling the model.

Step 7: Return the same kinds of HTTP responses: `200` for reads and updates, `201` for create, `404` when a plot is not found, and `400` when required data is missing.

### Part C: Model

Step 1: Open `src/models/userModel.js`.

Step 2: Use it as the pattern for `src/models/plotModel.js`.

Step 3: Keep the same model function pattern: select all, select by id, insert, update, delete.

Step 4: Replace the table import and column names so the file works with `plots` instead of `users`.

Step 5: Follow the same database query style used in the user model.

Step 6: Make sure `owner_id` and `seed_type` are the fields handled in create and update operations.

As you build `/plots`, compare each new file side by side with the matching users file. The users files already show the structure, naming pattern, and CRUD flow you should follow.

## 9) Test the Plots CRUD in Postman

Step 1: Start your app.

Step 2: Test each Plots endpoint in Postman.

Step 3: After each request, check the response.

Step 4: Refresh Drizzle Studio and confirm the `plots` table changed the way you expected.

After you finish the plots MVC files, test your plots CRUD in Postman to make sure it works.

Use this test flow:

1. `GET /plots` to confirm seeded plots are returned.
2. `GET /plots/:id` to confirm one plot can be loaded.
3. `POST /plots` with a JSON body to create a new plot.
4. `PUT /plots/:id` with a JSON body to update an existing plot.
5. `DELETE /plots/:id` to remove a plot.

After each request, check the response and refresh Drizzle Studio to confirm the `plots` table changed as expected.

## 10) Add a new Seeds entity by yourself

Step 1: Go back to your ERD.

Step 2: Add a new `Seeds` entity by yourself instead of copying a finished answer.

Step 3: Decide how `Seeds` connects to `Plots`.

Step 4: Update your schema so the database can store seed information properly.

This time you must design the ERD entity yourself, but your `Seeds` table needs to include at least these ideas:

1. A primary key column.
2. A column for the seed plant name.
3. A column for the seed color.
4. A structure that allows a plot to point to one seed record.

You may add extra useful columns if you want, but do not skip the core columns above.

## 11) Build the Seeds routes from requirements

Step 1: Create the new Seeds MVC files.

Step 2: Use the Users and Plots files as your examples.

Step 3: Build the routes based on the requirements below.

Your new files should include:

1. `src/routes/seedRoutes.js`.
2. `src/controllers/seedController.js`.
3. `src/models/seedModel.js`.

Route requirements for `/seeds`:

1. `GET /seeds` returns all seeds.
2. `GET /seeds/:seed_type` returns one seed by `seed_type`.
3. `POST /seeds` creates a new seed.
4. `PUT /seeds/:seed_type` updates one seed.
5. `DELETE /seeds/:seed_type` deletes one seed.

As you build these, follow the same MVC pattern you already used for Users and Plots:

1. Routes map the endpoints.
2. Controllers handle `req` and `res`.
3. Models run the database queries.

## 12) Seed and test the new Seeds table

Step 1: Add starter seed rows to `src/db/seed.js`.

Step 2: Run `npm run db` again.

Step 3: Open Drizzle Studio and confirm the `seeds` table exists and contains your sample rows.

Step 4: Test the `/seeds` CRUD routes in Postman.

Use the same CRUD test flow you used for Users and Plots.

## 13) Add a filter route using inner joins

Step 1: Add one more route to your project.

Step 2: Make that route find all plots that use a particular seed type.

Step 3: Return full information combined from all 3 tables: `Users`, `Plots`, and `Seeds`.

Step 4: Use `inner join` so only matching records are returned.

Route requirement:

1. Create a route that filters plots by `seed_type`.
2. The route should accept the `seed_type` value as part of the request.
3. The result should include joined information from `Users`, `Plots`, and `Seeds`.
4. Do not return only ids if you can return more meaningful joined data.

Use this route pattern:

```http
GET /plots/plant/:seed_type
```

Example:

If the request is:

```http
GET /plots/plant/2
```

One possible response shape could be:

```json
[
	{
		plot_id: 1,
		owner_id: 3,
		owner_name: 'Charlie',
		owner_email: 'charlie@example.com',
		seed_type: 2,
		seed_name: 'Carrot',
		seed_color: 'Orange'
	},
	{
		plot_id: 4,
		owner_id: 1,
		owner_name: 'Alice',
		owner_email: 'alice@example.com',
		seed_type: 2,
		seed_name: 'Carrot',
		seed_color: 'Orange'
	}
]
```

This shows the idea of the joined result: each row includes plot information, user information, and seed information together in one response.

## 14) Read the join guide before you code the join route

Step 1: Open [drizzle-orm.md](./drizzle-orm.md).

Step 2: Go to the `Joins and queries` section.

Step 3: Read the `Explicit join` example before you write your own query.

That article shows how `innerJoin(...)` works in Drizzle ORM. Use that pattern to build your 3-table query for users, plots, and seeds.

## 15) Test the joined route in Postman

Step 1: Run your app.

Step 2: Call your new filter route in Postman with a `seed_type` value that exists.

Step 3: Confirm the response includes combined data from the 3 joined tables.

Step 4: Try another `seed_type` value.

Step 5: Try a `seed_type` value that does not exist and decide what response your API should return.

Step 6: Refresh Drizzle Studio if needed to compare the API response with the stored data.

## 16) Run automated tests with the external test case repo

Step 1: Use this test case repository:

https://github.com/Mr-Siah-Class/testcases-practice-01

Step 2: Follow that repository setup so you can run the test suite against your API automatically.

Step 3: Read the README in that repo first to see exactly which routes are being tested.

Step 4: Run the test cases and use the results to confirm your endpoints and responses match the expected behavior.


