# BED Articles

A collection of backend development guides using Node.js and Express.

## Getting Started

Follow these guides in order:

| Step | Article | Description |
|------|---------|-------------|
| 1 | [Setup Environment](./setup-enviroment.md) | Pre-step: install tools, set Git credentials, clone repo, and learn `.gitignore`, `.env`, `node_modules`, and dependency workflow |
| 2 | [Express Server Setup](./express-server.md) | Create and run an Express server using JavaScript ES Modules, nodemon, and dotenv |
| 3 | [Express CRUD Routes](./express-routes.md) | Build GET, POST, PUT, and DELETE routes with explanations and Postman testing steps |
| 4 | [Express MVC Structure](./express-mvc.md) | Move CRUD logic into MVC folders (`routes`, `controllers`, `services`, `models`) using a `/users` example |
| 5 | [ERD Design, REST API, and MVC](./design-erd.md) | Design database schemas with dbdiagram.io, understand entities and relationships, and see how ERD design drives REST API and MVC structure (Pokemon example) |
| 6 | [LibSQL + Drizzle ORM Setup](./libsql-setup.md) | Replace in-memory arrays with a persistent LibSQL database and Drizzle ORM in the MVC project |

## Additional Reading

| Article | Description |
|---------|-------------|
| [Set Node Environment](./set-node-env.md) | Notes on setting and using environment variables with dotenv in a Node.js project |
| [Troubleshooting Guide](./troubleshooting.md) | Fix common Express issues such as `Cannot GET /`, stack traces, terminal errors, and debugging with `console.log` |
| [About Functions](./about-functions.md) | Learn JavaScript function basics, inputs/outputs, named functions, anonymous functions, arrow functions, and callbacks |
| [First-Class Functions and Arrow Functions in Express](./first-class-functions.md) | Learn why functions are values in Express, why arrow functions are standard in controllers, and how to write production-ready handlers with `const` exports |
| [About Callback and Async/Await](./about-async.md) | Learn callback flow, callback pitfalls, Promise basics, async/await syntax, and error handling in Express-style code |
| [Try/Catch in JavaScript and Express](./try-catch.md) | Learn try/catch fundamentals, graceful Express error responses, and how to debug hidden stack traces effectively |
| [HTTP Basics: Protocol, Messages, Methods, and Status Codes](./http-req.md) | Learn HTTP protocol fundamentals, message structure, methods, and status codes with backend-focused examples |
| [HTTP Variables in Express: req and res](./http-variables.md) | Learn req.query, req.params, req.body, res.locals, and how to read Express docs with methods vs properties in mind |
| [REST API Basics and Endpoint Design (With MVC)](./rest-api.md) | Learn what REST API means, how to design endpoints with MVC, and which route naming patterns are not REST-style |
| [Express Middleware Guide](./expresss-middleware.md) | Learn what middleware functions are, how `next()` works, and how built-in Express middleware like `express.json()` and `express.urlencoded()` are used |
