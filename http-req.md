# HTTP Basics: Protocol, Messages, Methods, and Status Codes

This guide introduces the core HTTP concepts every backend learner should know.

## 1) What is HTTP

HTTP stands for HyperText Transfer Protocol.

It is the communication rulebook used by clients and servers on the web.

Simple view:

1. Client sends an HTTP request.
2. Server processes it.
3. Server sends an HTTP response.

Example client and server:

1. Client: browser, Postman, frontend app, mobile app
2. Server: Express API, web server, backend service

## 2) What is a protocol

A protocol is a standard set of rules for communication.

For HTTP, rules define:

1. Message format
2. Allowed methods (GET, POST, etc.)
3. Status codes (200, 404, 500, etc.)
4. Header structure

Without protocol rules, client and server cannot reliably understand each other.

## 3) HTTP message structure

HTTP has two message types:

1. Request message (client -> server)
2. Response message (server -> client)

### Request message parts

1. Start line: method + path + HTTP version
2. Headers: metadata (content type, auth, etc.)
3. Body: optional payload (common in POST/PUT/PATCH)

Example request:

```http
POST /users HTTP/1.1
Host: localhost:3000
Content-Type: application/json

{
  "name": "Alice",
  "email": "alice@example.com"
}
```

### Response message parts

1. Status line: HTTP version + status code + reason phrase
2. Headers: metadata about response
3. Body: response data

Example response:

```http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "id": 1,
  "name": "Alice",
  "email": "alice@example.com"
}
```

## 4) HTTP methods (verbs)

Methods describe the intent of a request.

### Most common methods for APIs

1. GET: read data
2. POST: create new data
3. PUT: replace existing data
4. PATCH: partially update data
5. DELETE: remove data

Quick mapping:

| Method | Typical use | Example |
|---|---|---|
| GET | Read users | GET /users |
| GET | Read one user | GET /users/1 |
| POST | Create user | POST /users |
| PUT | Replace user | PUT /users/1 |
| PATCH | Partial update | PATCH /users/1 |
| DELETE | Delete user | DELETE /users/1 |

## 5) HTTP status codes

Status codes are how the server reports result of a request.

### Status code groups

1. 1xx: informational
2. 2xx: success
3. 3xx: redirection
4. 4xx: client error
5. 5xx: server error

### Common API status codes

| Code | Meaning | Typical API usage |
|---|---|---|
| 200 | OK | Successful GET/PUT/DELETE |
| 201 | Created | Successful POST |
| 204 | No Content | Success with no response body |
| 400 | Bad Request | Invalid input/body |
| 401 | Unauthorized | Missing/invalid auth |
| 403 | Forbidden | Authenticated but no permission |
| 404 | Not Found | Route/resource does not exist |
| 409 | Conflict | Duplicate or conflicting state |
| 422 | Unprocessable Content | Validation failed |
| 500 | Internal Server Error | Unhandled backend error |

## 6) How this maps to Express

In Express controllers, you usually do:

1. Read request data from req
2. Run logic/model
3. Send status + body

Example:

```js
export const getUserById = async (req, res) => {
	const payload = {
		id: req.params.id
	};

	const result = await selectUserById(payload);

	if (!result) {
		res.status(404).json({ message: 'User not found' });
		return;
	}

	res.status(200).json(result);
	return;
};
```

What learners should notice:

1. Request data comes from req.params, req.query, req.body.
2. Status code tells outcome clearly.
3. Response body gives useful API message/data.

## 7) Where to learn the official details

Use MDN as the reference source.

HTTP messages:

- https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Messages

HTTP status code reference:

- https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status

## 8) Practical checklist for students

Before finishing an endpoint, check:

1. Correct HTTP method used
2. Correct route path used
3. Correct status code returned
4. Response body is clear and useful
5. Error cases handled (400/404/500)

If these are correct, your API behavior is much easier to test and debug.
