# Try/Catch in JavaScript and Express

This guide explains how `try/catch` works, how to use it in Express for graceful API responses, and how to debug when error file/line details seem missing.

Use this together with the debugging guide in [troubleshooting.md](./troubleshooting.md).

## 1) What `try/catch` does

`try/catch` lets your program handle runtime errors without crashing the current request flow.

Basic shape:

```js
try {
	// code that might throw
} catch (error) {
	// handle the error
}
```

Why this matters in backend:

1. Your server should not crash because one request fails.
2. Clients should get clear HTTP responses.
3. You still need enough debug info to fix the root issue.

## 2) What `try/catch` catches (and what it does not)

`try/catch` catches:

1. Synchronous errors thrown inside the `try` block.
2. Async errors from awaited Promises (`await`) inside the `try` block.

`try/catch` does not catch:

1. Errors from async work that is not awaited.
2. Errors that happen outside the `try` scope.

Example:

```js
try {
	JSON.parse('{ invalid json }'); // caught
} catch (error) {
	console.error('Parse error:', error.message);
}
```

With async/await:

```js
try {
	const user = await findUserById(id); // rejection is caught
	return user;
} catch (error) {
	console.error('findUserById failed:', error.message);
}
```

## 3) Express: return graceful responses

A graceful response means:

1. Correct HTTP status code.
2. Clear, safe message for API client.
3. Internal details are logged for developers.

Simple controller pattern:

```js
export const getUserById = async (req, res) => {
	try {
		const user = await selectUserById({ id: req.params.id });

		if (!user) {
			res.status(404).json({ message: 'User not found' });
			return;
		}

		res.status(200).json(user);
		return;
	} catch (error) {
		// Log detailed error for debugging
		console.error('GET /users/:id failed');
		console.error(error);

		// Return safe message to client
		res.status(500).json({ message: 'Internal Server Error' });
		return;
	}
};
```

Why this is better than returning `error.message` directly:

1. Avoids leaking sensitive internals.
2. Keeps API response consistent.
3. Debug details still exist in server logs.

## 4) Better Express pattern: pass to error middleware

For larger apps, prefer central error middleware. In controllers, forward the error with `next(error)`.

Controller:

```js
export const getUserById = async (req, res, next) => {
	try {
		const user = await selectUserById({ id: req.params.id });

		if (!user) {
			res.status(404).json({ message: 'User not found' });
			return;
		}

		res.status(200).json(user);
		return;
	} catch (error) {
		next(error);
	}
};
```

Global error middleware (register after routes):

```js
app.use((error, req, res, next) => {
	console.error('Unhandled request error:', error);

	res.status(500).json({
		message: 'Internal Server Error'
	});
});
```

Official reference for this 4-argument error middleware signature:

- [Express Error Handling Guide](https://expressjs.com/en/guide/error-handling.html)

Benefits:

1. One place for logging.
2. One place for response format.
3. Cleaner controllers.

## 5) Why file/line can look "missing" after `catch`

A common beginner bug:

```js
catch (error) {
	res.status(500).json({ message: error.message });
}
```

If you only return `error.message`, you lose stack context in your terminal output.

The stack trace usually lives in:

1. `error.stack`
2. The full `error` object

So when debugging, always log the full error:

```js
catch (error) {
	console.error('Controller failed:', error);
	console.error('Stack:', error.stack);
	res.status(500).json({ message: 'Internal Server Error' });
}
```

This aligns with the workflow in [troubleshooting.md](./troubleshooting.md): read terminal errors first, start from the first app file frame, and fix root cause.

## 6) Debug flow when stack trace seems hidden

Use this repeatable process:

1. Reproduce with same request (same URL, method, body).
2. In `catch`, log full `error` and `error.stack`.
3. Read the first stack frame that points to your app file.
4. Add temporary logs before/after suspicious lines.
5. Compare expected vs actual values.
6. Fix one root issue.
7. Re-run same request.

## 7) Does `console.log` help? Yes, and here is why

Yes, it helps a lot during debugging.

`console.log` (or `console.error`) helps you verify:

1. Did this route/controller execute?
2. What are actual runtime values (`req.params`, `req.query`, `req.body`)?
3. Which line executes before failure?

Good style:

```js
console.log('Route hit: GET /users/:id');
console.log('params:', req.params);
console.log('query:', req.query);
```

For errors, prefer:

```js
console.error('getUserById failed:', error);
```

Why not only `error.message`:

1. `error.message` gives only text.
2. `error.stack` gives call path (file, line, call chain).
3. Full error object often contains more debug context.

Important production note:

1. Remove noisy debug logs after fixing.
2. Never log secrets (passwords, tokens, API keys).

## 8) Common mistakes to avoid

1. Catching error and doing nothing (silent failure).
2. Returning raw internal error details to clients.
3. Forgetting `return` after `res.status(...).json(...)` and accidentally sending multiple responses.
4. Not using centralized error middleware in bigger apps.

## 9) Mental model

1. `try`: risky code.
2. `catch`: graceful fallback for the client.
3. Logs: full details for developers.

If your API response is safe and your logs still show full stack trace, your error handling setup is healthy.
