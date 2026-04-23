# About Callback and Async/Await

This guide explains two core JavaScript concepts used heavily in backend development:

1. Callback functions
2. Async/Await

Use this after understanding basic functions in [about-functions.md](./about-functions.md).

## 1) Why this matters in backend

In backend apps, many tasks take time:

1. Reading from database
2. Calling external APIs
3. Reading/writing files

JavaScript does not want to freeze the whole server while waiting.

So we use async patterns to say:

1. Start this task
2. Continue other work
3. Handle result when ready

## 2) What is a callback

A callback is a function passed into another function, to be executed later.

Simple example:

```js
function processUser(userName, callback) {
	console.log('Processing user:', userName);
	callback(userName);
}

processUser('Alice', function (name) {
	console.log('Done for', name);
});
```

How it works:

1. `processUser` receives a function as input.
2. It decides when to run that function.
3. That input function is the callback.

## 3) Callback in async style

Older Node.js style often uses callback with `(error, result)`.

```js
function getUserFromDb(id, callback) {
	setTimeout(() => {
		if (!id) {
			callback(new Error('id is required'), null);
			return;
		}

		const user = { id: id, name: 'Alice' };
		callback(null, user);
	}, 300);
}

getUserFromDb(1, function (error, result) {
	if (error) {
		console.log('Error:', error.message);
		return;
	}

	console.log('User:', result);
});
```

## 4) Callback problem: callback hell

When many async steps depend on each other, nested callbacks become hard to read.

```js
step1((error1, result1) => {
	if (error1) return;

	step2(result1, (error2, result2) => {
		if (error2) return;

		step3(result2, (error3, result3) => {
			if (error3) return;
			console.log(result3);
		});
	});
});
```

This is one reason modern JavaScript uses Promise + async/await.

## 5) Promise in one minute

A Promise is an object that represents a future value.

It has 3 states:

1. Pending
2. Fulfilled
3. Rejected

Example:

```js
function getUserPromise(id) {
	return new Promise((resolve, reject) => {
		setTimeout(() => {
			if (!id) {
				reject(new Error('id is required'));
				return;
			}

			resolve({ id: id, name: 'Alice' });
		}, 300);
	});
}
```

## 6) What is async/await

`async/await` is cleaner syntax to handle Promises.

Rules:

1. `async` marks a function that can use `await`.
2. `await` pauses that function until Promise resolves or rejects.
3. Use `try/catch` for async error handling.

Example:

```js
async function run() {
	try {
		const result = await getUserPromise(1);
		console.log('User:', result);
	} catch (error) {
		console.log('Error:', error.message);
	}
}

run();
```

## 7) Callback vs async/await

Callback style:

```js
getUserFromDb(1, function (error, result) {
	if (error) {
		console.log(error.message);
		return;
	}

	console.log(result);
});
```

Async/await style:

```js
async function run() {
	try {
		const result = await getUserPromise(1);
		console.log(result);
	} catch (error) {
		console.log(error.message);
	}
}
```

Both are valid. Async/await is usually easier to read.

## 8) Express controller example (real backend usage)

This is why your controller functions become async when model/database calls are async.

```js
export const getUserById = async (req, res) => {
	const payload = {
		id: req.params.id
	};

	try {
		const result = await selectUserById(payload);

		if (!result) {
			res.status(404).json({ message: 'User not found' });
			return;
		}

		res.status(200).json(result);
		return;
	} catch (error) {
		res.status(500).json({ message: error.message });
		return;
	}
};
```

Why this pattern is important:

1. `await` waits for DB result.
2. `try/catch` handles runtime errors.
3. `return` prevents sending multiple responses.

## 9) Common beginner mistakes

1. Using `await` outside `async` function.
2. Forgetting `try/catch` around awaited DB/API calls.
3. Sending response without `return`, then sending another response later.
4. Mixing callback and async/await in confusing ways.

## 10) Mental model to remember

1. Callback: “Run this function later.”
2. Promise: “I will give you a value later.”
3. Async/Await: “Write async code in readable step-by-step style.”

If learners can explain this in plain words, they understand the core concept.
