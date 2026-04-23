# HTTP Variables in Express: req and res

This guide teaches the most important request/response variables in Express:

1. `req.query`
2. `req.params`
3. `req.body`
4. `res.locals`

It also explains how to read official Express documentation and understand methods vs properties.

## 1) Big picture: req and res

In Express route handlers:

1. `req` is the incoming request data.
2. `res` is how your server sends a response.

Example:

```js
app.get('/hello', (req, res) => {
	res.status(200).json({ message: 'Hello' });
});
```

## 2) `req.params` (path variables)

Use `req.params` to read dynamic values from URL path segments.

Route example:

```js
app.get('/users/:id', (req, res) => {
	const result = {
		userId: req.params.id
	};

	res.status(200).json(result);
	return;
});
```

If URL is `/users/42`, then:

1. `req.params.id` is `'42'`

Use cases:

1. Get one item by id
2. Update/delete item by id

## 3) `req.query` (URL query string)

Use `req.query` for filtering, sorting, paging, or search.

Route example:

```js
app.get('/users', (req, res) => {
	const result = {
		role: req.query.role,
		page: req.query.page,
		limit: req.query.limit
	};

	res.status(200).json(result);
	return;
});
```

If URL is `/users?role=admin&page=2&limit=10`, then:

1. `req.query.role` is `'admin'`
2. `req.query.page` is `'2'`
3. `req.query.limit` is `'10'`

Important:

1. Query values are usually strings.
2. Convert when needed (`Number(req.query.page)`).

## 4) `req.body` (request payload)

Use `req.body` for data sent in POST/PUT/PATCH requests.

You must enable JSON parser middleware first:

```js
app.use(express.json());
```

Route example:

```js
app.post('/users', (req, res) => {
	const payload = {
		name: req.body.name,
		email: req.body.email
	};

	res.status(201).json(payload);
	return;
});
```

If body is:

```json
{
	"name": "Alice",
	"email": "alice@example.com"
}
```

Then `req.body.name` is `Alice`.

## 5) Quick comparison

| Variable | Source | Typical use |
|---|---|---|
| `req.params` | URL path (`/users/:id`) | Identify one resource |
| `req.query` | URL query (`?page=1`) | Filter/sort/pagination |
| `req.body` | Request payload | Create/update data |

## 6) `res.locals` (per-request shared data)

`res.locals` is a temporary object for the current request only.

It is useful for passing data from middleware to later middleware/controllers.

Example:

```js
function authMiddleware(req, res, next) {
	res.locals.user = {
		id: 'u1',
		role: 'admin'
	};

	next();
}

app.get('/profile', authMiddleware, (req, res) => {
	const result = {
		currentUser: res.locals.user
	};

	res.status(200).json(result);
	return;
});
```

Why `res.locals` is useful:

1. Avoids repeating expensive work (decode token, load user).
2. Keeps data request-scoped.
3. Keeps controller clean.

## 7) Other important req/res info

Useful `req` properties:

1. `req.method` -> HTTP method (`GET`, `POST`, etc.)
2. `req.path` -> route path part
3. `req.headers` -> incoming headers
4. `req.ip` -> client IP (proxy setup affects this)

Useful `res` methods:

1. `res.status(code)` -> set status code
2. `res.json(data)` -> send JSON response
3. `res.send(textOrData)` -> send body
4. `res.set(name, value)` -> set response header

## 8) Methods vs properties (important for beginners)

In JavaScript objects:

1. Property = value/variable on object
2. Method = function on object

Examples with Express:

1. Property examples:
   - `req.params`
   - `req.query`
   - `req.body`
2. Method examples:
   - `res.status(200)`
   - `res.json({ ok: true })`

Easy rule:

1. If it has `()`, it is a method (function call).
2. If no `()`, it is a property (data).

## 9) How to read Express official documentation

Start here:

1. Main site: https://expressjs.com/
2. API reference: https://expressjs.com/en/4x/api.html

Best reading order for beginners:

1. Request object section (`req`)
2. Response object section (`res`)
3. Router section
4. Middleware guide

How to read each API item:

1. Name: Example `req.query`
2. Type: property or method
3. Input: arguments (for methods)
4. Output/behavior: what it returns/does
5. Example: copy and run quickly

Example doc reading checklist:

1. `req.query` -> property, object of query params
2. `req.params` -> property, object of route params
3. `res.status(code)` -> method, sets status and returns `res`
4. `res.json(body)` -> method, sends JSON response

## 10) Practice exercise

Build one route using all concepts:

1. Route: `GET /products/:id?preview=true`
2. Read:
   - `req.params.id`
   - `req.query.preview`
3. Add middleware that sets `res.locals.requestStart`
4. Return JSON with all values

If learners can explain where each value came from, they understand HTTP variables well.
