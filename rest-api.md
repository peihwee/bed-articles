# REST API Basics and Endpoint Design (With MVC)

This guide explains:

1. What a REST API is
2. How to design endpoints properly
3. How endpoint design maps to MVC
4. What route names do not follow REST style

## 1) What is a REST API

REST means Representational State Transfer.

In beginner backend terms, a REST API is:

1. Resource-focused URL design
2. HTTP methods used by intent (GET, POST, PUT, PATCH, DELETE)
3. Stateless requests (each request contains what server needs)
4. Standard HTTP status code usage

Resource means a thing in your system, for example:

1. users
2. products
3. orders

## 2) REST endpoint design mindset

Design URLs as nouns (resources), not verbs (actions).

Good REST style:

1. `GET /users`
2. `GET /users/:id`
3. `POST /users`
4. `PUT /users/:id`
5. `PATCH /users/:id`
6. `DELETE /users/:id`

Why this is better:

1. URL identifies resource
2. Method identifies action
3. API becomes predictable for everyone

## 3) REST + MVC: who does what

When designing with MVC in mind:

1. Route layer:
	- Defines REST path and HTTP method
	- Example: `router.get('/:id', getUserById)`
2. Controller layer:
	- Handles HTTP request/response
	- Builds payload and calls model
3. Model layer:
	- Handles data access
	- Query, insert, update, delete

Quick mapping example:

1. `GET /users/:id`
2. Route -> `getUserById`
3. Controller -> reads `req.params.id`, builds payload
4. Model -> `selectUserById(payload)`

## 4) REST route naming rules

Use these as default rules:

1. Use plural nouns for collection: `/users`
2. Use path param for one item: `/users/:id`
3. Do not put HTTP verbs in route path
4. Keep route lowercase and consistent
5. Use query string for filtering/searching, not new action routes

Good examples:

1. `GET /users?role=admin`
2. `GET /products?category=games&limit=10`

## 5) Route names that do NOT follow REST API style

| Not REST-style | Why not good | REST-style replacement |
|---|---|---|
| `GET /getUsers` | Verb in URL | `GET /users` |
| `POST /createUser` | Verb in URL | `POST /users` |
| `PUT /updateUser/:id` | Verb in URL | `PUT /users/:id` |
| `DELETE /deleteUser/:id` | Verb in URL | `DELETE /users/:id` |
| `POST /users/delete/:id` | Action route for deletion | `DELETE /users/:id` |
| `GET /user` | Inconsistent singular collection route | `GET /users` |

Important note:

Action-like routes can exist for non-CRUD domain actions (example: `/orders/:id/cancel`), but for standard CRUD resources, prefer REST naming first.

## 6) Example: users routes in REST + MVC style

### routes/userRoutes.js

```js
/* ----------------------------
	IMPORTS
------------------------------ */
import { Router } from 'express';
import {
	getUsers,
	getUserById,
	postUser,
	putUserById,
	deleteUserById
} from '../controllers/userController.js';

/* ----------------------------
	ROUTER SETUP
------------------------------ */

const router = Router();

/* ----------------------------
	REST ROUTE DEFINITIONS
------------------------------ */

router.get('/', getUsers);
router.get('/:id', getUserById);
router.post('/', postUser);
router.put('/:id', putUserById);
router.delete('/:id', deleteUserById);

/* ----------------------------
	EXPORT
------------------------------ */

export default router;
```

### index.js mount point

```js
app.use('/users', userRoutes);
```

Final public routes become:

1. `GET /users`
2. `GET /users/:id`
3. `POST /users`
4. `PUT /users/:id`
5. `DELETE /users/:id`

## 7) Status code alignment with REST

Recommended defaults:

1. GET success -> 200
2. POST created -> 201
3. PUT/PATCH success -> 200 (or 204)
4. DELETE success -> 200 (or 204)
5. Not found -> 404
6. Invalid input -> 400
7. Unexpected server error -> 500

## 8) Endpoint design checklist for learners

Before finalizing a route:

1. Is URL a resource noun (not a verb)?
2. Is HTTP method matching the action intent?
3. Is one item route using `/:id`?
4. Are controller and model responsibilities separated?
5. Is status code correct for success/error?

If these are yes, your endpoint design is likely REST-friendly and MVC-friendly.
