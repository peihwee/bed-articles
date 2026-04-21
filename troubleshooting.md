# Troubleshooting Guide

Use this guide when your Express app is not behaving as expected.

## 1) "Cannot GET /" - what it means

`Cannot GET /` means your server is running, but there is no matching `GET /` route.

Common causes:

1. You did not create `app.get('/')`.
2. You created a route, but with the wrong method (for example `POST /` instead of `GET /`).
3. You are calling the wrong URL or port.
4. Your route file is not imported/used.

## 2) Quick checklist for "Cannot GET /"

1. Confirm server is running:

```powershell
npm run dev
```

You should see something like:

```text
Server running on Port 3000
```

2. Confirm URL is correct in browser/Postman:

- `http://localhost:3000/`

3. Confirm route exists in code:

```js
app.get('/', (req, res) => {
	res.status(200).json({ message: 'GET / works' });
});
```

4. Restart server after code changes (especially if auto-reload did not trigger).

## 3) Read errors in terminal first

The terminal is your first debugger.

When an error happens, check:

1. The error message title (example: `ReferenceError`, `TypeError`).
2. The file and line number shown.
3. The stack trace (what called what).

Example:

```text
ReferenceError: userName is not defined
	at createUser (file:///project/index.js:25:15)
	at file:///project/index.js:42:3
	at Layer.handleRequest (.../node_modules/router/lib/layer.js:152:17)
	at next (.../node_modules/router/lib/route.js:157:13)
```

How to use this:

1. Start from the first app file frame (`index.js:25:15`).
2. Open that line and inspect the exact variable/function there.
3. Check the next app frame (`index.js:42:3`) to understand who called it.
4. Fix the root issue, then run again.

How to "step to the error" quickly:

1. Find the error type (`ReferenceError`, `TypeError`, etc.).
2. Read the first stack line that points to your file.
3. Use that as your primary fix location.
4. Use later lines only to understand call flow.

Which lines can you usually ignore first:

1. Frames from `node_modules/...`
2. Internal Node.js runtime frames

These lines are often framework internals (Express/router internals). They show path flow, but the bug is usually in your app file frame above them.

## 4) Basic debug flow (simple and repeatable)

1. Reproduce the error consistently.
2. Read terminal error top to bottom.
3. Identify exact file + line.
4. Add temporary logs around that area.
5. Run again and compare expected vs actual values.
6. Fix one thing at a time.
7. Re-test the same request.

## 5) Why `console.log` is important

`console.log` helps you trace program flow and inspect real runtime values.

Use it to answer:

1. Did this route execute?
2. What is inside `req.params`, `req.query`, `req.body`?
3. Is a variable undefined, null, or wrong type?

Example tracing in route:

```js
app.post('/users/:id', (req, res) => {
	console.log('Route hit: POST /users/:id');
	console.log('params:', req.params);
	console.log('query:', req.query);
	console.log('body:', req.body);

	res.status(200).json({ ok: true });
});
```

## 6) Good `console.log` habits

1. Add labels (`'params:'`, `'body:'`) so logs are readable.
2. Log before and after important logic blocks.
3. Remove or reduce noisy logs once issue is fixed.
4. Never log secrets in real projects (`password`, tokens, API keys).

## 6.1) Why use comma in `console.log` instead of `+`

Prefer this style when debugging:

```js
console.log('id:', id);
console.log('user:', user);
```

Instead of:

```js
console.log('id: ' + id);
console.log('user: ' + user);
```

Why comma is better:

1. Objects stay readable. With `+`, objects often print as `[object Object]`.
2. Data types are preserved better in console tools.
3. Less string-building mistakes (missing spaces, wrong concatenation).
4. Easier multi-value tracing in one line:

```js
console.log('params:', req.params, 'query:', req.query, 'body:', req.body);
```

Use `+` only when you intentionally want one final formatted string.

## 7) Extra checks when routes still fail

1. Ensure `app.use(express.json())` is added before routes that read `req.body`.
2. Ensure static/special routes (example `/search`) are above dynamic routes (example `/:id`).
3. Ensure you are using the correct HTTP method in Postman/browser.
4. Ensure only one server is using the same port.

## 8) Fast self-check command list

```powershell
node -v
npm -v
npm run dev
```

If the app starts, test quickly:

```text
GET http://localhost:3000/
```

---

If you are stuck, copy the full terminal error and the route code block together before asking for help. That gives enough context to debug quickly.
