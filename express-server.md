# Step-by-step guide: Set up an Express server

This guide uses JavaScript (ES Modules) and matches your package.json setup.

## 1) Create a new Node.js project

Open a terminal in your project folder and run:

```bash
npm init -y
```

## 2) Set author, type, and scripts with npm init tooling

Run these commands to update `package.json` without manually editing it:

```bash
npm pkg set author="Your Name"
npm pkg set type="module"
npm pkg set scripts.start="node index.js"
npm pkg set scripts.dev="nodemon index.js"
```

## 3) What is type="module" and why it is important

`type: "module"` in `package.json` tells Node.js to treat `.js` files as ES Modules (ESM).

Why this matters in your project:

1. You can use `import` and `export` syntax in `index.js`.
2. Your current server code uses `import express from 'express';`, which requires ESM mode in `.js` files.
3. Without it, Node may throw errors such as "Cannot use import statement outside a module".
4. It keeps your code aligned with modern JavaScript module standards.

Quick rule:

- If you use `import/export` in `.js`, keep `type` as `module`.
- If you use `require/module.exports`, use CommonJS instead (no `type: "module"`).

## 4) Install dependencies

Install exactly the dependencies you want:

```bash
npm install dotenv express nodemon
```

## 5) Verify package.json

Expected `package.json` to have the following:

```json
{
  "name": "folder-name",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node index.js",
    "dev": "nodemon index.js"
  },
  "keywords": [],
  "author": "Your Name",
  "license": "ISC",
  "type": "module",
  "dependencies": {
    "dotenv": "^17.4.2",
    "express": "^5.2.1",
    "nodemon": "^3.1.14"
  }
}
```

## 6) Create `index.js` (GET routes first)

Create file:

- `index.js`

Paste this simplified code first:

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
   ROUTES
------------------------------ */

// GET / — returns a simple message
app.get('/', (req, res) => {
	res.status(200).json({
		message: 'GET / works'
	});
});

// GET /search — reads query string, e.g. /search?name=bed
app.get('/search', (req, res) => {
	res.status(200).json({
		query: req.query
	});
});

// GET /:id — reads a dynamic URL segment, e.g. /123
app.get('/:id', (req, res) => {
	res.status(200).json({
		id: req.params.id
	});
});

/* ----------------------------
   START SERVER
------------------------------ */
app.listen(PORT, () => {
	console.log(`Server running on Port ${PORT}`);
});
```

## 7) (Optional) Create .env

Create `.env` in the project root if you want a custom port:

```env
PORT=3000
```

## 8) Run the server in development

```bash
npm run dev
```

Expected output:

```text
Server running on Port 3000
```

## 9) Test GET routes in the browser

Run `npm run dev`, then open these URLs in your browser:

1. `http://localhost:3000/`
2. `http://localhost:3000/123`
3. `http://localhost:3000/search?name=bed&role=student`

Expected behavior:

1. `/` returns a JSON message.
2. `/:id` returns the `id` from the URL.
3. `/search?...` returns the query string values from `req.query`.

## 10) Add POST with `req.body`

After GET routes work, add JSON middleware and a POST route.

Update `index.js` to this:

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
   ROUTES
------------------------------ */

// GET / — returns a simple message
app.get('/', (req, res) => {
	res.status(200).json({
		message: 'GET / works'
	});
});

// GET /search — reads query string, e.g. /search?name=bed
app.get('/search', (req, res) => {
	res.status(200).json({
		query: req.query
	});
});

// GET /:id — reads a dynamic URL segment, e.g. /123
app.get('/:id', (req, res) => {
	res.status(200).json({
		id: req.params.id
	});
});

// POST / — reads a JSON body sent in the request
app.post('/', (req, res) => {
	res.status(201).json({
		message: 'POST / works',
		body: req.body
	});
});

/* ----------------------------
   START SERVER
------------------------------ */
app.listen(PORT, () => {
	console.log(`Server running on Port ${PORT}`);
});
```

## 11) Install Postman (to test POST and request bodies)

Where to get Postman:

1. Go to https://www.postman.com/downloads/
2. Download Postman for Windows.
3. Run the installer and sign in (or continue with a free account).

Why you need Postman:

1. Browsers are easy for simple GET URL tests.
2. POST/PUT/PATCH/DELETE with JSON body are much easier in Postman.
3. You can set headers like `Content-Type: application/json` quickly.
4. You can save and reuse requests for debugging and team sharing.

## 12) Test POST in Postman

1. Open Postman and click `New` -> `HTTP Request`.
2. Set method to `POST`.
3. URL: `http://localhost:3000/`
4. Go to `Body` -> select `raw` -> choose `JSON`.
5. Paste:

```json
{
	"name": "bed",
	"role": "student"
}
```

6. Click `Send`.
7. Expected response:

```json
{
	"message": "POST / works",
	"body": {
		"name": "bed",
		"role": "student"
	}
}
```

## 13) Starting the server

You have two scripts set up, and they serve different purposes.

### Development mode

```bash
npm run dev
```

This uses `nodemon`, which watches your files and automatically restarts the server every time you save a change. You do not need to stop and restart manually while building your app.

> **Note:** nodemon only watches `.js`, `.mjs`, `.json`, and a few other code file extensions by default. Saving a `.env` file will **not** trigger a restart. If you change environment variables in `.env`, stop the server (`Ctrl + C`) and run `npm run dev` again manually.

### Production mode

```bash
npm start
```

This runs the server once with `node index.js` and does not watch for file changes. Use this when the code is final and you are serving real users.

## 14) Why some scripts need `run` and others do not

npm has a small set of built-in script names that it recognises directly:

| Command | Works without `run`? |
|---|---|
| `npm start` | Yes |
| `npm test` | Yes |
| `npm stop` | Yes |
| `npm restart` | Yes |

Any custom script name like `dev`, `build`, or `lint` requires the `run` keyword:

```bash
npm run dev    # custom name, needs run
npm run build  # custom name, needs run
npm start      # built-in, no run needed
npm test       # built-in, no run needed
```

Rule: if you made up the script name yourself, always add `run`.

---

## Next steps

Now that your server is running, learn how to build full CRUD routes (GET, POST, PUT, DELETE):

[express-routes.md](./express-routes.md)
