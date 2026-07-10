# Local Storage + DOM Guide (Using Login Page Example)

Continue this after [frontend-fetch.md](./frontend-fetch.md).

This guide teaches you how to:

1. Understand what DOM is.
2. Read user input from DOM elements.
3. Store login token in `localStorage`.
4. Use token for protected API requests.
5. Log out by clearing stored token.

---

## 1) What Is DOM?

DOM stands for **Document Object Model**.

Think of it as the browser's JavaScript view of your HTML page.

When this HTML is loaded:

```html
<input type="text" id="username" required>
```

JavaScript can access and control it through DOM methods:

```javascript
const usernameInput = document.getElementById("username");
console.log(usernameInput.value);
```

In simple words:

1. HTML creates elements.
2. Browser converts elements into DOM objects.
3. JavaScript reads or changes those objects.

---

## 2) Example Login HTML (DOM Source)

Use this structure in `login.html`:

```html
<main class="py-4">
	<div class="container">
		<div class="row justify-content-center mt-5">
			<div class="col-md-6 px-5">
				<h3 class="mb-4 text-center">Login</h3>
				<form id="loginForm">
					<div class="form-group pb-3">
						<label for="username">Username</label>
						<input type="text" class="form-control" id="username" required>
					</div>
					<div class="form-group pb-3">
						<label for="password">Password</label>
						<input type="password" class="form-control" id="password" required>
					</div>
					<button type="submit" class="btn btn-primary btn-block">Login</button>
				</form>
				<div id="warningCard" class="card border-danger mt-3 mb-3 d-none">
					<div class="card-body text-danger">
						<p id="warningText" class="card-text"></p>
					</div>
				</div>
			</div>
		</div>
	</div>
</main>
```

Important DOM IDs used by JavaScript:

1. `loginForm`
2. `username`
3. `password`
4. `warningCard`
5. `warningText`

---

## 3) Why We Use `DOMContentLoaded`

In this snippet, we start with:

```javascript
document.addEventListener("DOMContentLoaded", function () {
	// login logic here
});
```

This means: "Run my JavaScript only after the browser has finished building the page structure (DOM)."

Why this matters to you:

1. We want to access elements like `loginForm`, `username`, and `password` safely.
2. If JavaScript runs too early, `document.getElementById(...)` can return `null`.
3. `DOMContentLoaded` prevents that common beginner error.

---

## 4) Step-by-Step Teaching Walkthrough of `loginUser.js`

Use this exact snippet while explaining each step:

```javascript
document.addEventListener("DOMContentLoaded", function () {
	const callback = (responseStatus, responseData) => {
		console.log("responseStatus:", responseStatus);
		console.log("responseData:", responseData);
		if (responseStatus == 200) {
			// Check if login was successful
			if (responseData.token) {
				// Store the token in local storage
				localStorage.setItem("token", responseData.token);
				// Redirect or perform further actions for logged-in user
				window.location.href = "profile.html";
			}
		} else {
			warningCard.classList.remove("d-none");
			warningText.innerText = responseData.message;
		}
	};

	const loginForm = document.getElementById("loginForm");

	const warningCard = document.getElementById("warningCard");
	const warningText = document.getElementById("warningText");

	loginForm.addEventListener("submit", function (event) {
		console.log("loginForm.addEventListener");
		event.preventDefault();

		const username = document.getElementById("username").value;
		const password = document.getElementById("password").value;

		const data = {
			username: username,
			password: password,
		};
		// Perform login request
		fetchMethod(currentUrl + "/api/login", callback, "POST", data);

		// Reset the form fields
		loginForm.reset();
	});
});
```

What this code does step by step:

1. `document.addEventListener("DOMContentLoaded", ...)`
	Start only after DOM is ready.
2. `const loginForm = document.getElementById("loginForm")`
	Grab the form once so we can attach events to it.
3. `loginForm.addEventListener("submit", function (event) { ... })`
	Listen for the user clicking **Login**.
4. `event.preventDefault()`
	Stop the browser's default form behavior (page refresh).
5. `document.getElementById("username").value` and `...("password").value`
	Read what the user typed from DOM inputs.
6. `fetchMethod(currentUrl + "/api/login", callback, "POST", data)`
	Send login data to backend API.
7. Inside `callback`, check API result:
	If token exists, save it with `localStorage.setItem("token", ...)`.
8. `window.location.href = "profile.html"`
	Move user to protected page after successful login.
9. On error, show warning in DOM:
	`warningCard.classList.remove("d-none")` and set `warningText`.

Practice tip for you:
Trace one full login flow: typing input -> submit -> API response -> token storage -> redirect.

---

## 5) What Is `localStorage`?

`localStorage` is browser storage for key-value string data.

Main points for you:

1. Data stays after refresh.
2. Data stays after browser restart.
3. Data is per origin (same protocol + domain + port).
4. Values are stored as strings.

Basic commands:

```javascript
localStorage.setItem("token", "abc123");
const token = localStorage.getItem("token");
localStorage.removeItem("token");
localStorage.clear();
```

---

## 6) Beginner-Friendly Version of `loginUser.js`

This version keeps the same logic, but it is easier for you to read because warning behavior is grouped into small helper functions.

```javascript
document.addEventListener("DOMContentLoaded", function () {
	const loginForm = document.getElementById("loginForm");
	const warningCard = document.getElementById("warningCard");
	const warningText = document.getElementById("warningText");

	const showWarning = (message) => {
		warningText.innerText = message;
		warningCard.classList.remove("d-none");
	};

	const hideWarning = () => {
		warningText.innerText = "";
		warningCard.classList.add("d-none");
	};

	const callback = (responseStatus, responseData) => {
		console.log("responseStatus:", responseStatus);
		console.log("responseData:", responseData);

		if (responseStatus === 200 && responseData.token) {
			localStorage.setItem("token", responseData.token);
			window.location.href = "profile.html";
			return;
		}

		const message = responseData?.message || "Login failed. Please try again.";
		showWarning(message);
	};

	loginForm.addEventListener("submit", function (event) {
		event.preventDefault();
		hideWarning();

		const username = document.getElementById("username").value;
		const password = document.getElementById("password").value;

		const data = {
			username,
			password,
		};

		fetchMethod(currentUrl + "/api/login", callback, "POST", data);
		loginForm.reset();
	});
});
```

---

## 7) Script Loading Order in `login.html`

Place script tags before `</body>` like this:

```html
<script src="js/queryCmds.js" type="text/javascript"></script>
<script src="js/getCurrentURL.js" type="text/javascript"></script>
<script src="js/userNavbarToggle.js" type="text/javascript"></script>
<script src="js/loginUser.js" type="text/javascript"></script>
```

Why this works:

1. `loginUser.js` uses `fetchMethod` from `queryCmds.js`.
2. `loginUser.js` uses `currentUrl` from `getCurrentURL.js`.
3. So dependencies are loaded first.

---

## 8) How to Use Stored Token on Protected Pages

Example for `profile.js` or any protected page:

```javascript
document.addEventListener("DOMContentLoaded", function () {
	const token = localStorage.getItem("token");

	if (!token) {
		window.location.href = "login.html";
		return;
	}

	const callback = (status, data) => {
		console.log(status, data);
	};

	fetchMethod(currentUrl + "/api/profile", callback, "GET", null, token);
});
```

Key points for you:

1. Read token from `localStorage`.
2. If token missing, force login.
3. If token exists, send it as Bearer token in request headers.

---

## 9) Logout Example

```javascript
const logout = () => {
	localStorage.removeItem("token");
	window.location.href = "login.html";
};
```

You can connect this to a logout button click event.

---

## 10) Mini Classroom Checklist

After finishing this topic, you should be able to:

1. Explain DOM in one sentence.
2. Use `document.getElementById()` to read input values.
3. Handle form submit without page refresh.
4. Store token with `localStorage.setItem()`.
5. Read token with `localStorage.getItem()`.
6. Remove token on logout.
7. Explain why script order matters in HTML.

