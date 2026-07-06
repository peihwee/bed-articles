# Hashing Passwords With bcrypt Middleware in Express

Continue this after [libsql-setup.md](./libsql-setup.md).

When building login and registration features, you should never store plain text passwords.

Instead, you should:

1. Hash the password before saving it
2. Compare the login password against the stored hash later
3. Keep that logic in reusable middleware

This article shows you step by step how to install `bcrypt`, create `src/middleware/bcryptMiddleware.js`, test it with example routes, and adapt it to your own endpoints.

## 1) What you need first

Before you start, make sure you already have:

1. Node.js installed
2. An Express project created
3. `express.json()` enabled in your server so `req.body` works

Example:

```js
import express from 'express';

const app = express();

app.use(express.json());
```

If `express.json()` is missing, `req.body.password` may be `undefined`.

## 2) Install bcrypt

Open the terminal in your project and run:

```bash
npm install bcrypt
```

If your project does not already use ES modules, you may also need this in `package.json`:

```json
{
	"type": "module"
}
```

This article uses `import` syntax, so the examples assume ES modules are enabled.

## 3) Create the middleware folder and file

Create this file:

```text
src/middleware/bcryptMiddleware.js
```

Add this code:

```js
import bcrypt from 'bcrypt';

const saltRounds = 10;

export const comparePassword = (req, res, next) => {
	const callback = (err, isMatch) => {
		if (err) {
			console.error('Error bcrypt:', err);
			res.status(500).json(err);
		} else {
			if (isMatch) {
				next();
			} else {
				res.status(401).json({
					message: 'Wrong password',
				});
			}
		}
	};

	bcrypt.compare(req.body.password, res.locals.hash, callback);
};

export const hashPassword = (req, res, next) => {
	const callback = (err, hash) => {
		if (err) {
			console.error('Error bcrypt:', err);
			res.status(500).json(err);
		} else {
			res.locals.hash = hash;
			next();
		}
	};

	bcrypt.hash(req.body.password, saltRounds, callback);
};
```

## 4) Understand what this middleware does

There are two exported middleware functions:

### 4.1) `hashPassword`

This middleware:

1. Reads `req.body.password`
2. Uses `bcrypt.hash()` to turn it into a secure hash
3. Stores the result in `res.locals.hash`
4. Calls `next()` so the next handler can use the hash

This is useful for:

1. Register routes
2. Create-user routes
3. Reset-password routes

### 4.2) `comparePassword`

This middleware:

1. Reads the plain password from `req.body.password`
2. Reads the saved hash from `res.locals.hash`
3. Uses `bcrypt.compare()` to check if they match
4. Calls `next()` if the password is correct
5. Sends `401 Wrong password` if it is not correct

This is useful for:

1. Login routes
2. Protected actions that require password confirmation

## 5) Why use `res.locals`?

`res.locals` is a good place to pass data from one middleware to the next.

In this example:

1. `hashPassword` saves the generated hash in `res.locals.hash`
2. A later handler can send it back or store it in the database
3. `comparePassword` expects the existing hash to already be in `res.locals.hash`

That means one middleware can prepare data and the next one can use it.

## 6) Create example routes to test bcrypt

Create a route file such as:

```text
src/routes/exampleRoutes.js
```

Add this code:

```js
import { Router } from 'express';
import { hashPassword, comparePassword } from '../middleware/bcryptMiddleware.js';

const exampleRoutes = Router();

exampleRoutes.post('/bcrypt/hash', (req, res, next) => {
	if (!req.body) {
		return res.status(400).json({ message: 'Request body is required' });
	}

	if (!req.body.password) {
		return res.status(400).json({ message: 'Password is required' });
	}

	next();
}, hashPassword, (req, res) => {
	res.json({ hash: res.locals.hash });
});

exampleRoutes.post('/bcrypt/compare', (req, res, next) => {
	if (!req.body) {
		return res.status(400).json({ message: 'Request body is required' });
	}

	if (!req.body.password || !req.body.hash) {
		return res.status(400).json({ message: 'Password and hash are required' });
	}

	res.locals.hash = req.body.hash;

	next();
}, comparePassword, (req, res) => {
	res.json({ message: 'Password matches hash' });
});

export default exampleRoutes;
```

Important:

1. This article places the bcrypt file in `src/middleware/bcryptMiddleware.js`
2. That is why the import uses `../middleware/bcryptMiddleware.js`
3. If your project uses a folder named `middlewares` instead, keep the folder name and import path consistent everywhere

## 7) Connect the example routes to your server

In your server file, import and register the routes.

Example:

```js
import express from 'express';
import exampleRoutes from './src/routes/exampleRoutes.js';

const app = express();

app.use(express.json());
app.use('/api', exampleRoutes);

app.listen(3000, () => {
	console.log('Server running on port 3000');
});
```

If you use `/api`, the full endpoints become:

1. `POST /api/bcrypt/hash`
2. `POST /api/bcrypt/compare`

## 8) Test the hash route

Send a `POST` request to:

```text
/api/bcrypt/hash
```

With this JSON body:

```json
{
	"password": "mySecret123"
}
```

Example response:

```json
{
	"hash": "$2b$10$..."
}
```

What happened:

1. Your route checked that `req.body.password` exists
2. `hashPassword` hashed the password
3. The final route handler returned the hash from `res.locals.hash`

## 9) Test the compare route

Now send a `POST` request to:

```text
/api/bcrypt/compare
```

With this JSON body:

```json
{
	"password": "mySecret123",
	"hash": "$2b$10$..."
}
```

If the password matches the hash, the response is:

```json
{
	"message": "Password matches hash"
}
```

If the password is wrong, the response is:

```json
{
	"message": "Wrong password"
}
```

What happened:

1. The route validated that both `password` and `hash` exist
2. The route copied `req.body.hash` into `res.locals.hash`
3. `comparePassword` used `bcrypt.compare()` to check them
4. If they matched, Express moved to the final handler

## 10) How to integrate this into your own endpoints

The example routes are only for learning. In a real app, you usually use the middleware in registration and login endpoints.

### 10.1) Registration example

```js
router.post('/register', (req, res, next) => {
	if (!req.body?.password) {
		return res.status(400).json({ message: 'Password is required' });
	}

	next();
}, hashPassword, async (req, res) => {
	const newUser = {
		email: req.body.email,
		password: res.locals.hash,
	};

	// Save newUser to the database here

	res.status(201).json({
		message: 'User created',
		user: newUser,
	});
});
```

Flow:

1. User sends a plain password
2. `hashPassword` hashes it
3. You store `res.locals.hash` in the database, not the plain password

### 10.2) Login example

```js
router.post('/login', async (req, res, next) => {
	if (!req.body?.email || !req.body?.password) {
		return res.status(400).json({ message: 'Email and password are required' });
	}

	// Example: this user record was loaded from your database
	const user = await db.users.findOne({
		email: req.body.email,
	});

	if (!user) {
		return res.status(404).json({ message: 'User not found' });
	}

	res.locals.hash = user.password;
	res.locals.userId = user.id;

	next();
}, comparePassword, (req, res) => {
	res.json({
		message: 'Login successful',
		userId: res.locals.userId,
	});
});
```

Flow:

1. User sends email and password
2. You query the database for that user's record
3. You place the saved hashed password into `res.locals.hash`
4. `comparePassword` checks the login password against that stored hash
5. If it matches, the request continues

In a real project, `db.users.findOne(...)` could be replaced with your actual database code such as Prisma, Drizzle, Sequelize, Mongoose, or raw SQL.

## 11) Common beginner mistakes

### 11.1) Forgetting `express.json()`

If `req.body` is empty, make sure your app uses:

```js
app.use(express.json());
```

### 11.2) Saving the plain password instead of the hash

Do not save:

```js
req.body.password
```

Save:

```js
res.locals.hash
```

### 11.3) Using the wrong import path

If your file is here:

```text
src/middleware/bcryptMiddleware.js
```

then your route import should match that exact folder name.

### 11.4) Comparing against the wrong value

`comparePassword` expects the existing hashed password to already be stored in `res.locals.hash` before the middleware runs.

## 12) Summary

You now have a reusable bcrypt setup that:

1. Hashes passwords with `hashPassword`
2. Compares passwords with `comparePassword`
3. Passes data between middleware using `res.locals`
4. Can be dropped into registration, login, and password-check routes

If you understand the example test routes first, integrating the same middleware into your real endpoints becomes much easier.
