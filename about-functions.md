# About Functions

Functions are one of the most important parts of JavaScript.

A function is a reusable block of code that performs a task.

Why functions matter:

1. They let you reuse code instead of repeating it.
2. They make code easier to read.
3. They help you break one big problem into smaller steps.
4. They are used everywhere in Express, APIs, and callbacks.

## 1) Function basics

A function can:

1. Receive input
2. Do some work
3. Return output

Example:

```js
function add(a, b) {
	return a + b;
}
```

In this function:

1. `a` and `b` are inputs
2. `a + b` is the work
3. `return a + b` is the output

## 2) Input and output

### Input

The values inside the parentheses are called parameters.

```js
function greet(name) {
	console.log('Hello', name);
}
```

Here, `name` is the input parameter.

### Output

The `return` keyword sends a value out of the function.

```js
function multiply(x, y) {
	return x * y;
}

const result = multiply(2, 3);
console.log(result); // 6
```

Important:

1. `console.log()` shows something on screen.
2. `return` gives a value back to the caller.
3. A function may log something, return something, both, or neither.

## 3) Named function

A named function has a function name.

```js
function sayHello(name) {
	return 'Hello ' + name;
}
```

Why use named functions:

1. Easier to read
2. Easier to debug in stack traces
3. Good when the function has a clear responsibility

## 4) Anonymous function

An anonymous function has no direct name.

```js
const sayHello = function (name) {
	return 'Hello ' + name;
};
```

This function is anonymous, but it is stored in the variable `sayHello`.

Why use anonymous functions:

1. Useful when passing a function as a value
2. Common in callbacks

## 5) Arrow function

Arrow functions are a shorter way to write functions.

Long to short progression:

### Step 1: normal function expression

```js
const sayHello = function (name) {
	return 'Hello ' + name;
};
```

### Step 2: arrow function with parentheses and block body

```js
const sayHello = (name) => {
	return 'Hello ' + name;
};
```

### Step 3: remove parentheses for one parameter

```js
const sayHello = name => {
	return 'Hello ' + name;
};
```

### Step 4: shortest single-line version

```js
const sayHello = name => 'Hello ' + name;
```

What became shorter:

1. `function` keyword is removed.
2. Parentheses can be removed when there is only one parameter.
3. `{}` and `return` can be removed for a one-line return.

## 6) Why people use arrow functions

Arrow functions are common because:

1. They are shorter and cleaner.
2. They are convenient for small functions.
3. They are commonly used in modern JavaScript code.
4. They behave differently with `this`, which can avoid some confusion in callbacks.

Important note:

Arrow functions do **not** create their own `this`.
This is useful in some cases, but for beginners the main reason is usually cleaner syntax.

## 7) Function style comparison

These three examples do the same thing:

### Named function

```js
function square(num) {
	return num * num;
}
```

### Anonymous function

```js
const square = function (num) {
	return num * num;
};
```

### Arrow function

```js
const square = (num) => {
	return num * num;
};
```

## 8) What is a callback?

A callback is a function passed into another function.

That means:

1. One function is given as input to another function
2. The receiving function decides when to run it

Example:

```js
function runTask(callback) {
	console.log('Task started');
	callback();
}

runTask(function () {
	console.log('Callback executed');
});
```

Output:

```text
Task started
Callback executed
```

## 9) Callback with arrow function

The same callback can be written with an arrow function:

```js
function runTask(callback) {
	console.log('Task started');
	callback();
}

runTask(() => {
	console.log('Callback executed');
});
```

## 10) Why callbacks are important

Callbacks are used everywhere in JavaScript and Express.

Examples:

1. Route handlers in Express
2. Event listeners
3. Array methods like `map`, `filter`, `forEach`
4. Async operations

Express example:

```js
app.get('/users', (req, res) => {
	res.json({ message: 'Users route' });
});
```

Here:

1. `app.get()` is a function
2. `(req, res) => { ... }` is the callback
3. Express runs the callback when a request comes in

## 11) Quick examples with array callbacks

### `forEach`

```js
const numbers = [1, 2, 3];

numbers.forEach(num => {
	console.log(num);
});
```

### `map`

```js
const numbers = [1, 2, 3];

const doubled = numbers.map(num => num * 2);
console.log(doubled); // [2, 4, 6]
```

## 12) When to choose each style

Use named functions when:

1. The function has a clear job
2. You may reuse it in multiple places
3. You want better stack trace readability

Use anonymous or arrow functions when:

1. The function is short
2. It is used only once
3. It is passed as a callback

Use arrow functions when:

1. You want shorter syntax
2. You are writing small callback functions
3. You do not need a normal function `this`

## 13) Common beginner mistakes

1. Forgetting `return`

```js
const add = (a, b) => {
	a + b;
};
```

This returns `undefined` because there is no `return`.

Correct version:

```js
const add = (a, b) => {
	return a + b;
};
```

or

```js
const add = (a, b) => a + b;
```

2. Confusing `console.log` with `return`
3. Forgetting to call the function

```js
sayHello;
```

This only references the function.

Correct:

```js
sayHello('BED');
```

## 14) Summary

1. Functions help organize and reuse code.
2. Functions can take input and return output.
3. Named, anonymous, and arrow functions are different ways to write functions.
4. Arrow functions are common because they are shorter.
5. Callbacks are functions passed into other functions.
6. Express route handlers are callbacks.
