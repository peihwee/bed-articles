# Express CRUD Routes

This guide builds on [express-server.md](./express-server.md). Your server must already be running before testing these routes.

CRUD stands for:

| Letter | Action | HTTP Method |
|--------|--------|-------------|
| C | Create | `POST` |
| R | Read | `GET` |
| U | Update | `PUT` |
| D | Delete | `DELETE` |

---

## 1) What are routes?

A route is a combination of an HTTP method and a URL path that your server listens for. When a matching request comes in, Express runs the handler function you define.

```
app.METHOD(PATH, HANDLER)
```

- `METHOD` — HTTP verb: `get`, `post`, `put`, `delete`
- `PATH` — URL pattern: `/`, `/:id`, `/users`, etc.
- `HANDLER` — function that receives `req` and `res`

---

## 2) Set up the full CRUD example

Update `index.js` to this. It simulates a simple in-memory list of users:

```js
/* ----------------------------
   IMPORTS
------------------------------ */
import express from 'express';
import dotenv from 'dotenv';

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
   MOCK DATA
------------------------------ */

// Temporary in-memory store — resets every time the server restarts
let users = [
	{ id: 1, name: 'Alice' },
	{ id: 2, name: 'Bob' }
];

/* ----------------------------
   ROUTES — READ
------------------------------ */

// GET / — returns all users
app.get('/', (req, res) => {
	res.status(200).json(users);
});

// GET /:id — returns a single user by id
app.get('/:id', (req, res) => {
	const user = users.find(u => u.id === Number(req.params.id));

	if (!user) {
		return res.status(404).json({ message: 'User not found' });
	}

	res.status(200).json(user);
});

/* ----------------------------
   ROUTES — CREATE
------------------------------ */

// POST / — creates a new user from req.body
app.post('/', (req, res) => {
	const { name } = req.body;

	const newUser = {
		id: users.length + 1,
		name
	};

	users.push(newUser);

	res.status(201).json(newUser);
});

/* ----------------------------
   ROUTES — UPDATE
------------------------------ */

// PUT /:id — replaces the user's data with req.body
app.put('/:id', (req, res) => {
	const index = users.findIndex(u => u.id === Number(req.params.id));

	if (index === -1) {
		return res.status(404).json({ message: 'User not found' });
	}

	users[index] = { id: Number(req.params.id), ...req.body };

	res.status(200).json(users[index]);
});

/* ----------------------------
   ROUTES — DELETE
------------------------------ */

// DELETE /:id — removes a user by id
app.delete('/:id', (req, res) => {
	const index = users.findIndex(u => u.id === Number(req.params.id));

	if (index === -1) {
		return res.status(404).json({ message: 'User not found' });
	}

	users.splice(index, 1);

	res.status(200).json({ message: 'User deleted' });
});

/* ----------------------------
   START SERVER
------------------------------ */
app.listen(PORT, () => {
	console.log(`Server running on Port ${PORT}`);
});
```

---

## 3) GET — Read

Used to fetch data. Safe and read-only — it does not change anything on the server.

| Route | Purpose |
|-------|---------|
| `GET /` | Return all users |
| `GET /:id` | Return one user by id |

### Test in browser

Open these URLs directly:

- `http://localhost:3000/` — all users
- `http://localhost:3000/1` — user with id 1

### Test in Postman

1. Method: `GET`
2. URL: `http://localhost:3000/1`
3. Click `Send`

Expected response:

```json
{ "id": 1, "name": "Alice" }
```

---

## 4) POST — Create

Used to create a new resource. The data is sent in the **request body** as JSON.

### Test in Postman

1. Method: `POST`
2. URL: `http://localhost:3000/`
3. Go to `Body` → `raw` → `JSON`
4. Paste:

```json
{ "name": "Charlie" }
```

5. Click `Send`

Expected response:

```json
{ "id": 3, "name": "Charlie" }
```

---

## 5) PUT — Update

Used to update an existing resource. You target the resource by its id in the URL and send the new data in the **request body**.

> **Note:** PUT replaces the entire record. If you only send `{ "name": "..." }`, any other fields not included will be lost. Use PATCH if you only want to update specific fields.

### Test in Postman

1. Method: `PUT`
2. URL: `http://localhost:3000/1`
3. Go to `Body` → `raw` → `JSON`
4. Paste:

```json
{ "name": "Alice Updated" }
```

5. Click `Send`

Expected response:

```json
{ "id": 1, "name": "Alice Updated" }
```

---

## 6) DELETE — Delete

Used to remove a resource. You identify it by its id in the URL. No request body is needed.

### Test in Postman

1. Method: `DELETE`
2. URL: `http://localhost:3000/2`
3. Click `Send`

Expected response:

```json
{ "message": "User deleted" }
```

---

## 7) Common HTTP status codes (quick reference)

### Success (`2xx`)

| Code | Meaning | Common API usage |
|------|---------|------------------|
| `200` | OK | Successful GET, PUT, DELETE |
| `201` | Created | Successful POST (new resource created) |
| `204` | No Content | Successful request with no response body (often DELETE/PUT) |

### Client errors (`4xx`)

| Code | Meaning | Common API usage |
|------|---------|------------------|
| `400` | Bad Request | Missing/invalid request body, params, or query |
| `401` | Unauthorized | Missing or invalid authentication token |
| `403` | Forbidden | Authenticated, but no permission to access resource |
| `404` | Not Found | Route does not exist or resource id not found |
| `409` | Conflict | Duplicate resource / state conflict |
| `422` | Unprocessable Entity | Validation failed (format is correct but data is invalid) |
| `429` | Too Many Requests | Rate limit exceeded |

### Server errors (`5xx`)

| Code | Meaning | Common API usage |
|------|---------|------------------|
| `500` | Internal Server Error | Unexpected server-side error |
| `502` | Bad Gateway | Upstream dependency returned invalid response |
| `503` | Service Unavailable | Server temporarily overloaded/maintenance |
| `504` | Gateway Timeout | Upstream service took too long to respond |

---

## Next step

Now continue to structure this CRUD app using MVC:

[express-mvc.md](./express-mvc.md)
