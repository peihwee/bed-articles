# JWT Middleware in Express

This article continues from the password hashing article.

In the previous guide, you learned how to:

1. Hash a password before saving it
2. Compare a login password against a stored hash

The next step is creating a JWT after login so the client can prove it is authenticated on later requests.

In this article, you will learn how to:

1. Install the required packages
2. Create `src/middleware/jwtMiddleware.js`
3. Configure JWT environment variables
4. Update `exampleRoutes.js`
5. Test token generation and token verification
6. Reuse the middleware in your own login and protected routes

## 1) What JWT is used for

JWT stands for JSON Web Token.

A JWT is commonly used after a user logs in successfully.

Typical flow:

1. The user logs in with email and password
2. Your server checks the password
3. Your server creates a JWT
4. The client stores that token
5. The client sends the token in the `Authorization` header on later requests
6. Your server verifies the token before allowing access

This means:

1. `generateToken` is useful after login
2. `verifyToken` is useful for protected routes

## 2) What you need first

Before starting this article, make sure you already have:

1. Node.js installed
2. An Express project created
3. `express.json()` enabled
4. The bcrypt setup from the previous article completed

Example server setup:

```js
import express from 'express';

const app = express();

app.use(express.json());
```

## 3) Install the packages

Open the terminal in your project and install the JWT packages:

```bash
npm install jsonwebtoken dotenv
```

This article also uses ES modules, so if needed your `package.json` should include:

```json
{
	"type": "module"
}
```

## 4) Add your JWT environment variables

Create a `.env` file in the root of your project.

Example:

```env
JWT_SECRET_KEY=your_super_secret_key_here
JWT_EXPIRES_IN=15m
JWT_ALGORITHM=HS256
```

What these values mean:

1. `JWT_SECRET_KEY` is the secret used to sign and verify tokens
2. `JWT_EXPIRES_IN` controls how long the token lasts
3. `JWT_ALGORITHM` defines the signing algorithm

For learning, `HS256` is a common and simple choice.

Important:

1. Do not commit your real `.env` secrets to version control
2. If `JWT_SECRET_KEY` is missing, token generation and verification will fail

## 5) Create the middleware file

Create this file:

```text
src/middleware/jwtMiddleware.js
```

Add this code:

```js
import dotenv from 'dotenv';
dotenv.config();

import jwt from 'jsonwebtoken';

const secretKey = process.env.JWT_SECRET_KEY;
const tokenDuration = process.env.JWT_EXPIRES_IN;
const tokenAlgorithm = process.env.JWT_ALGORITHM;

export const generateToken = (req, res, next) => {
	const payload = {
		userId: res.locals.userId,
		timestamp: new Date(),
	};

	const options = {
		algorithm: tokenAlgorithm,
		expiresIn: tokenDuration,
	};

	const callback = (err, token) => {
		if (err) {
			console.error('Error jwt:', err);
			res.status(500).json(err);
		} else {
			res.locals.token = token;
			next();
		}
	};

	jwt.sign(payload, secretKey, options, callback);
};

export const sendToken = (req, res, next) => {
	res.status(200).json({
		message: res.locals.message,
		token: res.locals.token,
	});
};

export const verifyToken = (req, res, next) => {
	const authHeader = req.headers.authorization;

	if (!authHeader || !authHeader.startsWith('Bearer ')) {
		return res.status(401).json({ error: 'Invalid token' });
	}

	const token = authHeader.substring(7);

	if (!token) {
		return res.status(401).json({ error: 'No token provided' });
	}

	const callback = (err, decoded) => {
		if (err) {
			return res.status(401).json({ error: 'Invalid token' });
		}

		res.locals.userId = decoded.userId;
		res.locals.tokenTimestamp = decoded.timestamp;
		next();
	};

	jwt.verify(token, secretKey, callback);
};
```

## 6) Understand what each middleware function does

There are three exported functions.

### 6.1) `generateToken`

This middleware:

1. Reads `res.locals.userId`
2. Builds a JWT payload
3. Uses your `.env` settings to sign the token
4. Stores the generated token in `res.locals.token`
5. Calls `next()` so another handler can return the token

This is usually used after:

1. Login succeeds
2. Registration succeeds
3. Password verification succeeds

### 6.2) `sendToken`

This middleware:

1. Reads `res.locals.token`
2. Sends the token back as JSON

This is useful when you want a small reusable response step after token generation.

### 6.3) `verifyToken`

This middleware:

1. Reads the `Authorization` header
2. Checks that it starts with `Bearer `
3. Extracts the token
4. Verifies the token with your secret key
5. Stores decoded data in `res.locals`
6. Calls `next()` if the token is valid

This is usually used on protected routes.

## 7) Why use `res.locals` here too?

Just like the bcrypt article, this JWT middleware uses `res.locals` to pass data between middleware functions.

Examples:

1. A login route can store the database user's id in `res.locals.userId`
2. `generateToken` reads that id and creates a token
3. `sendToken` returns the token from `res.locals.token`
4. `verifyToken` decodes the token and stores the `userId` again for later handlers

This keeps each middleware focused on one job.

## 8) Update `exampleRoutes.js`

This article continues from the bcrypt example routes.

Use this file:

```text
src/routes/exampleRoutes.js
```

Update it to:

```js
import { Router } from 'express';
import { hashPassword, comparePassword } from '../middleware/bcryptMiddleware.js';
import { generateToken, sendToken, verifyToken } from '../middleware/jwtMiddleware.js';

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

exampleRoutes.post('/jwt/generate', (req, res, next) => {
	if (!req.body) {
		return res.status(400).json({ message: 'Request body is required' });
	}

	if (!req.body.userId) {
		return res.status(400).json({ message: 'User ID is required' });
	}

	res.locals.userId = req.body.userId;
	res.locals.message = 'Token generated successfully';

	next();
}, generateToken, sendToken);

exampleRoutes.post('/jwt/verify', (req, res, next) => {
	if (!req.headers || !req.headers.authorization) {
		return res.status(400).json({ message: 'Authorization header is required' });
	}

	next();
}, verifyToken, (req, res) => {
	res.json({ message: 'Token is valid', userId: res.locals.userId });
});

export default exampleRoutes;
```

Important:

1. This article places both middleware files in `src/middleware`
2. That is why the imports use `../middleware/...`
3. If your project uses `middlewares` instead, keep all folder names and imports consistent
4. `sendToken` returns `res.locals.message`, so the route sets that message before calling `generateToken`

## 9) Connect the routes in your server

If you already followed the bcrypt article, this part should look familiar.

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

The JWT endpoints will now be:

1. `POST /api/jwt/generate`
2. `POST /api/jwt/verify`

## 10) Test the generate token route

Send a `POST` request to:

```text
/api/jwt/generate
```

With this JSON body:

```json
{
	"userId": 123
}
```

Example response:

```json
{
	"message": "Token generated successfully",
	"token": "eyJ..."
}
```

What happened:

1. The route checked that `userId` exists
2. The route stored `userId` in `res.locals.userId`
3. `generateToken` created a signed JWT
4. `sendToken` returned the token as JSON

## 11) Test the verify token route

Now send a `POST` request to:

```text
/api/jwt/verify
```

Add this header:

```text
Authorization: Bearer your_token_here
```

If you are testing in Postman, you can add the bearer token in either of these places:

1. Open the request and click the `Authorization` tab
2. Change the type to `Bearer Token`
3. Paste the token value from `/api/jwt/generate` into the `Token` field

Postman will automatically build this header for you:

```text
Authorization: Bearer your_token_here
```

You can also add it manually in the `Headers` tab by creating an `Authorization` header and pasting the full `Bearer ...` value yourself.

Example response:

```json
{
	"message": "Token is valid",
	"userId": 123
}
```

What happened:

1. The route checked that the `Authorization` header exists
2. `verifyToken` extracted the token from `Bearer ...`
3. `verifyToken` checked the signature and expiration
4. The decoded `userId` was stored in `res.locals.userId`
5. The final route handler returned the result

## 12) How to use this after login

In a real application, you usually generate the JWT right after the password has been verified.

Example login flow:

```js
router.post('/login', async (req, res, next) => {
	if (!req.body?.email || !req.body?.password) {
		return res.status(400).json({ message: 'Email and password are required' });
	}

	const user = await db.users.findOne({
		email: req.body.email,
	});

	if (!user) {
		return res.status(404).json({ message: 'User not found' });
	}

	res.locals.hash = user.password;
	res.locals.userId = user.id;
	res.locals.message = 'Login successful';

	next();
}, comparePassword, generateToken, sendToken);
```

Flow:

1. The route loads the user record from the database
2. It stores the saved password hash in `res.locals.hash`
3. `comparePassword` checks the plain password against that hash
4. If the password is correct, `generateToken` creates the JWT
5. `sendToken` returns the token to the client

This is where the bcrypt article and the JWT article connect together.

## 13) How to protect your own routes

Once a client has a token, you can protect routes with `verifyToken`.

Example:

```js
router.get('/profile', verifyToken, (req, res) => {
	res.json({
		message: 'Profile loaded',
		userId: res.locals.userId,
	});
});
```

This means:

1. The client must send a valid token
2. `verifyToken` checks it first
3. The route can use `res.locals.userId` to load that user's data from the database

## 14) Common beginner mistakes

### 14.1) Forgetting to create `.env`

If the secret key or expiry settings are missing, JWT signing and verification may fail.

### 14.2) Forgetting `Bearer ` in the header

This middleware expects:

```text
Authorization: Bearer your_token_here
```

Not just the raw token by itself.

### 14.3) Not setting `res.locals.userId` before `generateToken`

`generateToken` expects `res.locals.userId` to already exist.

### 14.4) Returning `res.locals.message` without setting it

`sendToken` sends this:

```js
res.locals.message
```

So set that message earlier in the route if you want a clean response.

### 14.5) Mixing `middleware` and `middlewares` folders

Pick one folder name and keep the imports consistent.

## 15) Summary

You now have reusable JWT middleware that:

1. Creates a token with `generateToken`
2. Sends it with `sendToken`
3. Verifies it with `verifyToken`
4. Works well with the bcrypt login flow from the previous article

Once password checking and token generation are both working, you have the foundation for a real authentication flow in Express.

The client can then send that token in future requests, usually in the `Authorization` header.

This lets your server identify the user without asking for the password again on every request.
