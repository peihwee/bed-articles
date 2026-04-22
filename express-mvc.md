# Express MVC Guide

Continue this after [express-routes.md](./express-routes.md).

## 1) Why teams use MVC in industry

MVC means Model, View, Controller.

For API projects, the most common practical split is:

1. Route: defines URL and method.
2. Controller: handles request/response.
3. Model: handles data access and array/database logic.

Why this is industry practice:

1. Clear separation of concerns: each file has one responsibility.
2. Easier teamwork: multiple developers can work on different layers safely.
3. Easier testing: controller logic and model logic can be tested separately.
4. Easier scaling: adding features does not make one giant `index.js` file.
5. Easier maintenance: bugs are faster to locate because code is organized.

## 2) Folder structure example (`/users`)

Use this structure:

```text
project-root/
├─ index.js
├─ src/
│  ├─ routes/
│  │  └─ userRoutes.js
│  ├─ controllers/
│  │  └─ userController.js
│  └─ models/
│     └─ userModel.js
├─ package.json
└─ .env
```

What each file does:

1. `routes/userRoutes.js`: maps endpoints like `GET /users`.
2. `controllers/userController.js`: reads request and returns response using HTTP-style names like `getUsers`, `getUserById`, `postUser`.
3. `models/userModel.js`: stores user array and data logic using SQL-style names like `select`, `insert`, `update`, `delete`.
4. `index.js`: app setup and route registration.

## 3) Create folders and files

From project root:

```powershell
mkdir src
mkdir src\routes
mkdir src\controllers
mkdir src\models
ni index.js
ni src\routes\userRoutes.js
ni src\controllers\userController.js
ni src\models\userModel.js
```

If your `package.json` currently starts from another file, update scripts to use root `index.js`:

```powershell
npm pkg set scripts.start="node index.js"
npm pkg set scripts.dev="nodemon index.js"
```

## 4) Code the `/users` example

### `src/models/userModel.js`

```js
/* ----------------------------
   DATA STORE
------------------------------ */

// Temporary in-memory store (replace with DB later)
const users = [
	{ id: 1, name: 'Alice', email: 'alice@example.com' },
	{ id: 2, name: 'Bob', email: 'bob@example.com' }
];

/* ----------------------------
   READ FUNCTIONS
------------------------------ */

export const selectUsers = () => {
	const result = users;
	return result;
};

export const selectUserById = (payload) => {
	const result = users.find(u => u.id === Number(payload.id));
	return result ?? null;
};

/* ----------------------------
   CREATE FUNCTION
------------------------------ */

export const insertUser = (payload) => {
	const result = {
		id: users.length + 1,
		name: payload.name,
		email: payload.email
	};

	users.push(result);
	return result;
};

/* ----------------------------
   UPDATE FUNCTION
------------------------------ */

export const updateUser = (payload) => {
	const index = users.findIndex(u => u.id === Number(payload.id));
	if (index === -1) return null;

	users[index] = {
		id: Number(payload.id),
		...users[index],
		...payload
	};

	const result = users[index];
	return result;
};

/* ----------------------------
   DELETE FUNCTION
------------------------------ */

export const deleteUser = (payload) => {
	const index = users.findIndex(u => u.id === Number(payload.id));
	if (index === -1) return false;

	users.splice(index, 1);
	const result = true;
	return result;
};
```

### `src/controllers/userController.js`

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

export const getUsers = (req, res) => {
	const result = selectUsers();

	res.status(200).json(result);
	return;
};

export const getUserById = (req, res) => {
	const payload = {
		id: req.params.id
	};

	const result = selectUserById(payload);
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

export const postUser = (req, res) => {
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

	const result = insertUser(payload);
	res.status(201).json(result);
	return;
};

/* ----------------------------
   UPDATE HANDLER
------------------------------ */

export const putUserById = (req, res) => {
	const payload = {
		...req.body,
		id: req.params.id
	};

	const result = updateUser(payload);
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

export const deleteUserById = (req, res) => {
	const payload = {
		id: req.params.id
	};

	const result = deleteUserModel(payload);
	if (!result) {
		res.status(404).json({ message: 'User not found' });
		// return stops function execution to prevent sending a second response later
		return;
	}

	res.status(200).json({ message: 'User deleted' });
	return;
};
```

### `src/routes/userRoutes.js`

```js
/* ----------------------------
   IMPORTS
------------------------------ */
import { Router } from 'express';
import {
	getUsers,
	getUserById,
	postUser,
	putUserById,
	deleteUserById
} from '../controllers/userController.js';

/* ----------------------------
   ROUTER SETUP
------------------------------ */

const router = Router();

/* ----------------------------
   ROUTE DEFINITIONS
------------------------------ */

router.get('/', getUsers);
router.get('/:id', getUserById);
router.post('/', postUser);
router.put('/:id', putUserById);
router.delete('/:id', deleteUserById);

/* ----------------------------
   EXPORT
------------------------------ */

export default router;
```

### `index.js`

```js
/* ----------------------------
   IMPORTS
------------------------------ */
import express from 'express';
import dotenv from 'dotenv';
import userRoutes from './src/routes/userRoutes.js';

/* ----------------------------
   APP SETUP
------------------------------ */

dotenv.config();

const app = express();
const PORT = process.env.PORT || 3000;

/* ----------------------------
   MIDDLEWARE
------------------------------ */

// Parses incoming JSON request bodies into req.body
app.use(express.json());

/* ----------------------------
   ROUTES
------------------------------ */

// Mount all user routes under /users
app.use('/users', userRoutes);

// Health/check route for quick verification
app.get('/', (req, res) => {
	res.status(200).json({ message: 'MVC app is running' });
});

/* ----------------------------
   START SERVER
------------------------------ */

app.listen(PORT, () => {
	console.log(`Server running on Port ${PORT}`);
});
```

## 5) Test `/users` endpoints

Run server:

```powershell
npm run dev
```

Quick tests:

1. `GET http://localhost:3000/users`
2. `GET http://localhost:3000/users/1`
3. `POST http://localhost:3000/users` with body:

```json
{
	"name": "Charlie",
	"email": "charlie@example.com"
}
```

4. `PUT http://localhost:3000/users/1` with body:

```json
{
	"name": "Alice Updated"
}
```

5. `DELETE http://localhost:3000/users/2`

## 6) How this helps when your app grows

Without MVC:

1. One file becomes very long.
2. Routes, validation, and data logic are mixed together.
3. Changes can break unrelated features.

With MVC:

1. Routes stay clean and readable.
2. Controllers focus only on HTTP behavior and use HTTP-style names.
3. Models keep array/database logic in one place and can mirror SQL-style names.
4. Refactoring to real database models is easier later.

## 7) Why return after sending a response

In Express, each request must receive exactly one response.

When you call `res.status(...).json(...)`, the response is already sent.
If your function keeps running and tries to send another response, Express throws an error.

Common error message:

```text
Error [ERR_HTTP_HEADERS_SENT]: Cannot set headers after they are sent to the client
```

What this means:

1. You already sent one response.
2. Code continued running.
3. Another `res.status(...).json(...)` was attempted.

Wrong pattern (can send 2 responses):

```js
export const getUserById = (req, res) => {
	const result = selectUserById({ id: req.params.id });

	if (!result) {
		res.status(404).json({ message: 'User not found' });
	}

	// This still runs even when user is not found
	res.status(200).json(result);
};
```

Correct pattern (safe):

```js
export const getUserById = (req, res) => {
	const result = selectUserById({ id: req.params.id });

	if (!result) {
		res.status(404).json({ message: 'User not found' });
		return;
	}

	res.status(200).json(result);
	return;
};
```

Teaching rule for learners:

1. Send response.
2. Return immediately.
3. This prevents double-response bugs and runtime crashes.

## 8) What `...req.body` means

`...req.body` uses JavaScript object spread syntax.

In controllers, it is commonly used to copy all fields from `req.body` into a new object.

Example syntax:

```js
const payload = {
	...req.body,
	id: req.params.id
};
```

What this does:

1. `...req.body` expands all key-value pairs from the incoming request body.
2. `id: req.params.id` adds (or overrides) the `id` in the final payload.
3. The final object is passed to the model as one structured input.

Visual object example:

```js
const req = {
	params: { id: '7' },
	body: {
		id: '999',
		name: 'Alice Updated',
		email: 'alice.updated@example.com'
	}
};

// Safe order: route id wins
const payloadSafe = {
	...req.body,
	id: req.params.id
};

console.log(payloadSafe);
// {
//   id: '7',
//   name: 'Alice Updated',
//   email: 'alice.updated@example.com'
// }
```

Why order matters:

1. In object literals, later keys overwrite earlier keys.
2. This is safe:

```js
const payload = {
	...req.body,
	id: req.params.id
};
```

3. This is risky because body `id` can overwrite route `id`:

```js
const payload = {
	id: req.params.id,
	...req.body
};

console.log(payload);
// {
//   id: '999',
//   name: 'Alice Updated',
//   email: 'alice.updated@example.com'
// }
```

Teaching rule for learners:

1. Use spread to copy request body fields quickly.
2. Put trusted values (like route `id`) after spread so they win.
3. Pass one payload object to models for consistent function signatures.
