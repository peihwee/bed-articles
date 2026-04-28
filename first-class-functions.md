# First-Class Functions and Arrow Functions in Express

Continue this after [about-functions.md](./about-functions.md).

This guide teaches you how functions work as values in Express, why arrow functions are the standard, and how to write production-ready controllers.

## Official references (read these first)

Before or after this guide, read the official MDN references:

1. [MDN Glossary: First-class Function](https://developer.mozilla.org/en-US/docs/Glossary/First-class_Function)
2. [MDN JavaScript Reference: Arrow function expressions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)

Why this matters for learners:

1. MDN gives precise definitions, so your fundamentals are accurate.
2. MDN explains edge cases that tutorials often skip (especially arrow function `this`, `arguments`, and constructor behavior).
3. When debugging real projects, official docs help you verify behavior quickly instead of guessing.
4. Building a habit of checking primary references makes you more independent as a developer.

## 1) What is a first-class function?

In JavaScript, you should think of functions as values.

That means a function can be:

1. Saved in a variable
2. Passed into another function
3. Returned from another function

This idea is called a **first-class function**.

You will see this idea everywhere in Express.

Example:

```js
router.post('/users', postUser);
```

`postUser` is a function value being passed to `router.post`.

## 2) Should the `function` keyword be avoided in Express?

Short answer: do not treat `function` as "bad".

Both are valid:

```js
function postUser(req, res) {
	// ...
}
```

```js
const postUser = (req, res) => {
	// ...
};
```

So why do many teams use arrow functions more often?

1. Consistency: one style across controllers, models, and helpers
2. Modern style: common in current JavaScript codebases
3. Less `this` confusion: arrow functions do not create their own `this`
4. Easy composition: fits well with callbacks and inline function passing

Important point for you:

Use one team style and stay consistent.
Consistency is usually more important than "winning" a syntax debate.

## 3) Why first-class + arrow is common in Express

In Express, you pass handlers as values all the time:

```js
router.get('/users', getUsers);
router.post('/users', postUser);
```

Arrow + `const` is common for this because:

1. The handler is treated like a value
2. `const` prevents reassigning the handler accidentally
3. The pattern is easy for juniors to scan quickly

Example:

```js
export const getUsers = (req, res) => {
	res.status(200).json([]);
	return;
};
```

## 4) Evaluation of `postUser` code

Example code:

```js
export const postUser = (req, res) => {
		const { name, email } = req.body;
		if (!name || !email) {
				res.status(400).json({ message: 'name and email are required' });
				// return stops function execution to prevent sending a second response later
				return;
		}

		const payload = {
				name,
				email
		};

		const result = insertUser(payload);
		res.status(201).json(result);
		return;
};
```

Verdict: this is a good controller for beginner level.

What is good about this pattern:

1. Clear input validation (`name` and `email` required)
2. Correct error status (`400`) for bad input
3. Uses early return to avoid double response bug
4. Sends success response with status `201` for create
5. Uses a payload object before passing to model (clean design)

This is the **industry-standard** way to write Express controllers in 2026. It cleanly separates your logic (the controller) from your routing table.

### 4.1. The "Clean Code" Win

By assigning the arrow function to a `const`, you follow **Separation of Concerns**. Instead of having a "fat" router file with hundreds of lines of logic, your router becomes a clean table of contents:

```js
router.get("/", getUsers);
router.post("/", postUser);
router.put("/:id", putUserById);
```

### 4.2. Technical Breakdown

* **Variable Scope:** Using `const` ensures the handler reference cannot be reassigned elsewhere in your code, providing "read-only" safety for your logic.
* **Lexical `this`:** Since it is an arrow function, it does not create its own `this` context. In Express, this is usually fine because the framework relies on the `req` and `res` objects passed in.
* **Reusability:** You define the function once and pass the reference. This lets you reuse the same handler logic across multiple routes or export it to tests.

### 4.3. Comparison Table

| Feature | Inline: `router.get("/", (req, res) => {})` | Extracted: `router.get("/", postUser)` |
| :--- | :--- | :--- |
| **Readability** | Good for tiny, 1-line logic. | Excellent for real apps. |
| **Testability** | Hard to unit test the logic in isolation. | **Easy** to export `postUser` and test it. |
| **Reusability** | Impossible to reuse. | High. |

### 4.4. Stack Trace Clarity

The only minor consideration with `const controller = (req, res) => {}` involves **debugging**.

* **The Issue:** Because the function is technically anonymous (assigned to a variable), some older debuggers might show it as `anonymous` in a stack trace.
* **The Fix:** Modern engines infer the name from the variable. However, if you want absolute clarity in production logs, some teams use named function expressions:

```js
export const postUser = function postUser(req, res) {
  // ...
};
```

This is optional and rarely needed in modern production stacks.

### 4.5. What can improve for production

1. **Add centralized error handling:** Wrap controller logic in `try/catch` and forward unexpected errors to middleware for consistent `500` responses.
2. **Use request validation middleware:** Tools like `zod`, `joi`, or `express-validator` keep validation rules reusable and avoid repeating checks in each controller.
3. **Add uniqueness/business checks:** Example: reject duplicate email with `409 Conflict` when required by business rules.
4. **Add logging and tracing fields:** Structured logs and request IDs help teams debug failures quickly in staging and production.
5. **Add tests:** Unit tests for controller behavior and integration tests for route responses are expected in most professional codebases.

## 5) Should you name arrow functions with `const` for export?

This is a good question, and you may ask it often.

This is common and good:

```js
export const postUser = (req, res) => {
	// ...
};
```

Why teams like it:

1. Clear exported API (`postUser`, `getUsers`, `putUserById`)
2. Prevents reassignment because of `const`
3. Matches common style in modern Express tutorials and teams

Also valid:

```js
export function postUser(req, res) {
	// ...
}
```

Key differences for you:

1. `function` declarations are hoisted
2. `const` arrow functions are not usable before declaration
3. For controllers, both are fine if the team is consistent

## 6) Should all code be arrow functions for standardization?

Use this practical rule:

1. For controllers, routes, callbacks, small helpers: arrow style is great
2. For class methods that need dynamic `this`: prefer regular method/function style
3. For a codebase standard: pick one default and document exceptions

Simple team standard example:

1. Default: `export const name = (...) => {}`
2. Exception: use `function` when you need hoisting or dynamic `this`
3. Never mix styles randomly in the same folder

## 7) Final answer for you

1. `function` is not wrong.
2. Arrow + first-class style is popular because it is consistent and easy to pass around in Express.
3. Your `postUser` function is already good and safe.
4. Yes, exporting named `const` arrow handlers is a strong standard for Express projects.
5. Standardize by defaulting to arrow functions, with clear exceptions.

**Next step:** Apply this pattern to your controllers and organize them in a dedicated `src/controllers/` folder for scalability. See [express-mvc.md](./express-mvc.md) for the full folder structure and MVC guide.
