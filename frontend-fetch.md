# Frontend Fetch Guide: Build a Page That Calls an API and Renders DOM

Continue this after [jwtwebtoken.md](./jwtwebtoken.md).

This guide teaches you how to:

1. Build a simple frontend page.
2. Set up `getCurrentURL.js`.
3. Set up `queryCmds.js`.
4. Use a `showAllPokemon.js` example to understand DOM rendering with `fetch`.

Demonstration site used in this step-by-step guide: https://bed-final-example.vercel.app/index.html

---

## 1) Learning Goals

By the end, you should be able to explain this flow:

1. Browser loads HTML.
2. JavaScript calculates the API base URL.
3. JavaScript sends a GET request with `fetch`.
4. Callback receives JSON response.
5. DOM is updated with new elements.

---

## 2) Project File Setup

Create this structure in your project root (frontend is inside `public`, same level as `src`):

```text
project-root/
	index.js
	src/
		...backend files...
	public/
		index.html
		js/
			getCurrentURL.js
			queryCmds.js
			showAllPokemon.js
```

---

## 3) Setup Static Files in Root index.js

In your root `index.js`, mount API routes first, then serve `public` as static:

```javascript
//////////////////////////////////////////////////////
// SETUP ROUTES
//////////////////////////////////////////////////////
const mainRoutes = require('./routes/mainRoutes');
app.use('/api', mainRoutes);

//////////////////////////////////////////////////////
// SETUP STATIC FILES
//////////////////////////////////////////////////////
app.use('/', express.static('public'));
```

What this means for you:

1. Requests starting with `/api` go to backend route handlers.
2. Requests like `/`, `/js/queryCmds.js`, `/css/style.css` are served from the `public` folder.

---

## 4) Build Basic HTML (public/index.html)

Start with a container where JavaScript can inject cards.

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8" />
	<meta name="viewport" content="width=device-width, initial-scale=1.0" />
	<title>Pokemon List</title>
	<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet" />
</head>
<body>
	<main class="container py-4">
		<h1 class="mb-4">All Pokemon</h1>
		<div id="pokemonList" class="row"></div>
	</main>

	<script src="js/getCurrentURL.js"></script>
	<script src="js/queryCmds.js"></script>
	<script src="js/showAllPokemon.js"></script>
</body>
</html>
```

Important teaching point:

- Script order matters.
- `showAllPokemon.js` uses `currentUrl` and `fetchMethod`, so it must be loaded after `getCurrentURL.js` and `queryCmds.js`.

---

## 5) Step 1 Script: getCurrentURL.js

This script builds the base URL dynamically from the browser address.

```javascript
//const currentUrl = "http://" + window.location.hostname + ":" + window.location.port;
const currentUrl = window.location.protocol + "//" + window.location.host;
console.log("currentUrl:", currentUrl);
```

What to understand:

1. `window.location.protocol` gives `http:` or `https:`.
2. `window.location.host` gives domain + port (example: `localhost:3000`).
3. Combining them creates a reusable base URL for API calls.

Example result:

```text
http://localhost:3000
```

Why `getCurrentURL.js` is useful:

1. It avoids hardcoding one server URL in many files.
2. The same frontend code works in different environments:
	- Local: `http://localhost:3000`
	- Staging: `https://staging.example.com`
	- Production: `https://bed-final-example.vercel.app`
3. If domain or port changes, you do not need to rewrite every API call.
4. It reduces simple mistakes like calling the wrong host or wrong port.

Without `currentUrl`, you might write this repeatedly:

```javascript
fetch("http://localhost:3000/api/pokemon/dex");
```

Then they deploy and forget to change all URLs.

With `currentUrl`, you can write this instead:

```javascript
fetchMethod(currentUrl + "/api/pokemon/dex", callback);
```

This is easier to maintain and safer for real projects.

---

## 6) Step 2 Script: queryCmds.js

This script creates reusable API helper functions. Start with `fetchMethod` first so you can focus on one request style.

```javascript
//=====================================================================================
// FETCH METHOD
// This function uses the fetch API to make a request to the server.
//=====================================================================================
function fetchMethod(url, callback, method = "GET", data = null, token = null) {

	console.log("fetchMethod: ", url, method, data, token);

	const headers = {};

	if (data) {
		headers["Content-Type"] = "application/json";
	}

	if (token) {
		headers["Authorization"] = "Bearer " + token;
	}

	let options = {
		method: method.toUpperCase(),
		headers: headers,
	};

	if (method.toUpperCase() !== "GET" && data !== null) {
		options.body = JSON.stringify(data);
	}

	fetch(url, options)
		.then((response) => {
			if (response.status == 204) {
				callback(response.status, {});
			} else {
				response.json().then((responseData) => callback(response.status, responseData));
			}
		})
		.catch((error) => console.error(`Error from ${method} ${url}:`, error));
}
```

What to explain section by section:

1. Function parameters:
	 - `url`: API endpoint.
	 - `callback`: Function to run after response arrives.
	 - `method`: HTTP method (`GET`, `POST`, etc).
	 - `data`: Request payload for non-GET requests.
	 - `token`: Optional auth token.
2. Headers are built only when needed.
3. `options` object is what `fetch` needs.
4. For non-GET methods, body must be JSON string.
5. `fetch` is asynchronous and returns a Promise.
6. Response is passed to callback so DOM logic stays separate.

Tip:

- Keep request logic and UI logic in different files.
- You will usually learn faster when one function does one job.

---

## 7) Step 3 Script: showAllPokemon.js

This script calls the API and renders cards to DOM.

```javascript
const callback = (responseStatus, responseData) => {
	console.log("responseStatus:", responseStatus);
	console.log("responseData:", responseData);

	const pokemonList = document.getElementById("pokemonList");
	responseData.forEach((pokemon) => {
		const displayItem = document.createElement("div");
		displayItem.className = "col-xl-2 col-lg-3 col-md-4 col-sm-6 col-xs-12 p-3";
		displayItem.innerHTML = `
			<div class="card">
					<img src="https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/${pokemon.dex_num}.png" class="card-img-top" alt="Pokemon Image">
					<div class="card-body">
							<h5 class="card-title">${pokemon.name}</h5>
							<p class="card-text">
									ID: ${pokemon.id} <br>
									ATK: ${pokemon.atk} <br>
									DEF: ${pokemon.def} <br>
									HP: ${pokemon.hp} <br>
									Type 1: ${pokemon.type1} <br>
									Type 2: ${pokemon.type2} <br>
									Owner ID: ${pokemon.owner_id} <br>
							</p>
							<a href="singlePlayerInfo.html?player_id=${pokemon.owner_id}" class="btn btn-primary">View Owner</a>
					</div>
			</div>
			`;
		pokemonList.appendChild(displayItem);
	});
};

fetchMethod(currentUrl + "/api/pokemon/dex", callback);
```

---

## 8) DOM + Fetch Walkthrough (The Key Lesson)

Use this sequence:

1. Browser runs:

	 ```javascript
	 fetchMethod(currentUrl + "/api/pokemon/dex", callback);
	 ```

2. `fetchMethod` sends request to server.
3. Server returns JSON array of Pokemon.
4. Callback receives `responseData`.
5. `responseData.forEach(...)` loops through each Pokemon.
6. For each Pokemon:
	 - Create a new `<div>` with `document.createElement`.
	 - Fill content with template literal.
	 - Add it to page using `appendChild`.
7. Result: cards appear inside `#pokemonList`.

Simple mental model:

- API gives data.
- JavaScript turns data into HTML.
- DOM displays HTML.

---

## 9) DOM and innerHTML Explained

### What is the DOM?

DOM means Document Object Model.

Think of it as a live tree of your HTML page that JavaScript can read and change.

- `document.getElementById("pokemonList")` gets one DOM node.
- `document.createElement("div")` creates a new DOM node.
- `pokemonList.appendChild(displayItem)` attaches that node to the page.

When you append a node, the browser updates what users see.

### What is `innerHTML`?

`innerHTML` lets you insert HTML markup as a string inside an element.

In this guide:

```javascript
displayItem.innerHTML = `
	<div class="card">
		<h5>${pokemon.name}</h5>
		<p>ATK: ${pokemon.atk}</p>
	</div>
`;
```

This means:

1. Build a chunk of HTML text.
2. Put that text inside `displayItem`.
3. Then append `displayItem` to `pokemonList`.

### Why you may like `innerHTML`

1. Fast to write.
2. Easy to read for beginners.
3. Good for simple card/list rendering.

### Important safety note

If data comes from users, avoid inserting it directly with `innerHTML` because of XSS risk.

Safer beginner pattern for text:

```javascript
const title = document.createElement("h5");
title.textContent = pokemon.name;
displayItem.appendChild(title);
```

Use this rule:

- Trusted template/demo data: `innerHTML` is okay for learning.
- Untrusted user input: prefer `textContent` and `createElement`.

---

## 10) Common Mistakes You May Make

1. Wrong script order in HTML.
2. Wrong container id (must match `pokemonList`).
3. API URL typo (for example `/api/pokemon/dex`).
4. Server not running.
5. Trying to use response before Promise resolves.

Debug checklist:

1. Check browser console for errors.
2. Log `currentUrl`.
3. Log `responseStatus` and `responseData`.
4. Confirm network request in browser devtools.

---

## 11) Summary

You should now understand how to connect:

1. Dynamic base URL from browser (`public/js/getCurrentURL.js`).
2. Reusable request helper (`public/js/queryCmds.js`).
3. DOM rendering callback (`public/js/showAllPokemon.js`).
4. Static frontend delivery from `public` via root `index.js`.

That is the core frontend pattern used in many real projects: fetch data, then render UI.
