# Express Middleware Guide

Middleware is one of the most important ideas in Express.

## 1) What is middleware?

A middleware function is a function that runs between the request coming in and the final response being sent.

It can:

1. Read the request
2. Change the request or response object
3. Run logic like logging or validation
4. Stop the request and send a response early
5. Pass control to the next middleware or route handler

In Express, middleware usually looks like this:

```js
const middlewareFunction = (req, res, next) => {
	console.log('Middleware is running');
	next();
};
```

## 2) What are `req`, `res`, and `next` in a callback?

In Express, the function you pass into `app.use()`, `app.get()`, `app.post()`, and other route methods is a callback.

Example middleware callback:

```js
const middlewareFunction = (req, res, next) => {
	console.log('Middleware is running');
	next();
};
```

Example route callback:

```js
app.get('/users', (req, res) => {
	res.json({ message: 'Users route' });
});
```

What each parameter means:

1. `req` = request object
	It contains incoming request data such as:
	- `req.params`
	- `req.query`
	- `req.body`
	- `req.method`
	- `req.url`
2. `res` = response object
	You use it to send data back to the client:
	- `res.json(...)`
	- `res.send(...)`
	- `res.status(...)`
3. `next` = function that tells Express to continue to the next middleware or route handler

Important:

1. Route callbacks usually use `(req, res)`.
2. Middleware callbacks usually use `(req, res, next)`.
3. If middleware should continue, call `next()`.
4. If middleware sends a response, it usually should not call `next()` after that.

If you do not call `next()` and also do not send a response, the request can hang.

## 3) Simple middleware flow

```js
const app = express();

const logMiddleware = (req, res, next) => {
	console.log('Request received');
	next();
};

app.get('/', logMiddleware, (req, res) => {
	res.json({ message: 'Home route' });
});
```

Flow:

1. Request comes in
2. `logMiddleware` runs first
3. `next()` moves to the route handler
4. Route handler sends response

## 4) App-level middleware

If you want middleware to run for many routes, use `app.use()`.

```js
app.use((req, res, next) => {
	console.log(req.method, req.url);
	next();
});
```

This will run for every incoming request.

## 5) Route-level middleware

If you want middleware for only one route, attach it to that route.

```js
const checkMiddleware = (req, res, next) => {
	console.log('Checking this route only');
	next();
};

app.get('/users', checkMiddleware, (req, res) => {
	res.json({ message: 'Users route' });
});
```

## 6) Built-in Express middleware

Express provides built-in middleware functions that are used very often.

## 6.1) `express.json()`

This reads JSON request bodies and puts the parsed result into `req.body`.

```js
app.use(express.json());
```

Why you need it:

1. Without it, `req.body` may be `undefined` for JSON requests.
2. It is commonly used for `POST`, `PUT`, and `PATCH` APIs.

Example:

```js
app.use(express.json());

app.post('/users', (req, res) => {
	console.log(req.body);
	res.json({ body: req.body });
});
```

If you send this JSON:

```json
{
	"name": "Alice"
}
```

Then `req.body` becomes:

```js
{ name: 'Alice' }
```

## 6.2) `express.urlencoded()`

This reads form data sent with `application/x-www-form-urlencoded`.

```js
app.use(express.urlencoded({ extended: true }));
```

Why you need it:

1. Useful when HTML forms submit data.
2. It also places parsed values into `req.body`.

Example form data:

```text
name=Alice&role=student
```

After parsing, `req.body` becomes:

```js
{ name: 'Alice', role: 'student' }
```

About `extended`:

1. `extended: true` allows richer/nested data structures.
2. `extended: false` is simpler parsing.
3. For most beginner Express apps, `true` is a safe default.

## 6.3) Using both together

In many apps, you will see both:

```js
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
```

This means the app can read:

1. JSON request bodies
2. HTML form request bodies

## 7) Middleware order matters

Order is very important in Express.

Example:

```js
app.use(express.json());

app.post('/users', (req, res) => {
	res.json(req.body);
});
```

This works because `express.json()` runs before the route.

Wrong order:

```js
app.post('/users', (req, res) => {
	res.json(req.body);
});

app.use(express.json());
```

This may fail because the route runs before the JSON middleware.

## 8) Middleware that stops the request

Middleware does not always need to call `next()`.
It can send a response early.

```js
const blockMiddleware = (req, res, next) => {
	return res.status(403).json({ message: 'Forbidden' });
};

app.get('/admin', blockMiddleware, (req, res) => {
	res.json({ message: 'Admin page' });
});
```

Here the route handler will never run because middleware already sent a response.

## 9) Common middleware use cases

Middleware is often used for:

1. Logging requests
2. Parsing request body
3. Authentication
4. Validation
5. Error handling
6. Setting shared values with `res.locals`

Example using `res.locals`:

```js
const addMessage = (req, res, next) => {
	res.locals.message = 'Hello from middleware';
	next();
};

app.get('/', addMessage, (req, res) => {
	res.json({ message: res.locals.message });
});
```

## 10) Common beginner mistakes

1. Forgetting `next()` in middleware that should continue
2. Using `req.body` without `express.json()`
3. Putting middleware after the route instead of before it
4. Sending a response and then calling `next()` unnecessarily

## 11) Quick example with `express.json()` and `express.urlencoded()`

```js
import express from 'express';

const app = express();

app.use(express.json());
app.use(express.urlencoded({ extended: true }));

app.post('/submit', (req, res) => {
	console.log('body:', req.body);
	res.status(200).json({
		message: 'Data received',
		body: req.body
	});
});

app.listen(3000, () => {
	console.log('Server running on Port 3000');
});
```

## 12) Summary

1. Middleware runs before the final route response.
2. It can read, modify, block, or pass the request.
3. `next()` moves to the next middleware or route handler.
4. `express.json()` parses JSON body data.
5. `express.urlencoded()` parses form body data.
6. Middleware order matters a lot in Express.
