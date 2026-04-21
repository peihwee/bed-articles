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
	USER MODEL
------------------------------ */

const users = [
	{ id: 1, name: 'Alice', email: 'alice@example.com' },
	{ id: 2, name: 'Bob', email: 'bob@example.com' }
];

export const selectUsers = () => users;

export const selectUserById = id => users.find(u => u.id === Number(id));

export const insertUser = payload => {
	const newUser = {
		id: users.length + 1,
		name: payload.name,
		email: payload.email
	};

	users.push(newUser);
	return newUser;
};

export const updateUser = (id, payload) => {
	const index = users.findIndex(u => u.id === Number(id));
	if (index === -1) return null;

	users[index] = { id: Number(id), ...users[index], ...payload };
	return users[index];
};

export const deleteUser = id => {
	const index = users.findIndex(u => u.id === Number(id));
	if (index === -1) return false;

	users.splice(index, 1);
	return true;
};
```

### `src/controllers/userController.js`

```js
/* ----------------------------
   USER CONTROLLER
------------------------------ */
import {
	selectUsers,
	selectUserById,
	insertUser,
	updateUser,
	deleteUser as deleteUserModel
} from '../models/userModel.js';

export const getUsers = (req, res) => {
	res.status(200).json(selectUsers());
};

export const getUserById = (req, res) => {
	const user = selectUserById(req.params.id);
	if (!user) return res.status(404).json({ message: 'User not found' });

	res.status(200).json(user);
};

export const postUser = (req, res) => {
	const { name, email } = req.body;
	if (!name || !email) {
		return res.status(400).json({ message: 'name and email are required' });
	}

	const user = insertUser({ name, email });
	res.status(201).json(user);
};

export const putUserById = (req, res) => {
	const user = updateUser(req.params.id, req.body);
	if (!user) return res.status(404).json({ message: 'User not found' });

	res.status(200).json(user);
};

export const deleteUserById = (req, res) => {
	const ok = deleteUserModel(req.params.id);
	if (!ok) return res.status(404).json({ message: 'User not found' });

	res.status(200).json({ message: 'User deleted' });
};
```

### `src/routes/userRoutes.js`

```js
/* ----------------------------
   USER ROUTES
------------------------------ */
import { Router } from 'express';
import {
	getUsers,
	getUserById,
	postUser,
	putUserById,
	deleteUserById
} from '../controllers/userController.js';

const router = Router();

router.get('/', getUsers);
router.get('/:id', getUserById);
router.post('/', postUser);
router.put('/:id', putUserById);
router.delete('/:id', deleteUserById);

export default router;
```

### `index.js`

```js
/* ----------------------------
   APP ENTRY
------------------------------ */
import express from 'express';
import dotenv from 'dotenv';
import userRoutes from './src/routes/userRoutes.js';

dotenv.config();

const app = express();
const PORT = process.env.PORT || 3000;

app.use(express.json());
app.use('/users', userRoutes);

app.get('/', (req, res) => {
	res.status(200).json({ message: 'MVC app is running' });
});

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
