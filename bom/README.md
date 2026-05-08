- Category: JavaScript
- Difficulty: Intermediate
- Related: dom, event-loop

### BOM — The Browser Object Model

The **Browser Object Model (BOM)** is the collection of objects the browser exposes to JavaScript for controlling the browser environment — tabs, URLs, history, screen, timers, and storage. The root of everything is the **`window`** object. Every global variable you declare (`var x`) and every built-in (`setTimeout`, `alert`) is actually a property of `window`.

**Analogy**
If the DOM is the content inside your house (furniture, rooms), the BOM is the house itself and everything around it — the address (location), the lock history (history), the neighborhood info (navigator), the windows and doors (screen), and the security system timer (setTimeout/setInterval).

---

### 1. The Window Object — Root of Everything

**Theory**: `window` is the global object in browsers. You can omit `window.` when calling any of its properties — `setTimeout` is `window.setTimeout`, `alert` is `window.alert`.

**Working Flow**
![flow-chart](flow-chart.png)

**Example**
```javascript
// These are all the same — window is implicit
setTimeout(fn, 1000);         // window.setTimeout(fn, 1000)
console.log(innerWidth);      // window.innerWidth
console.log(window.location.href); // current URL

// window dimensions
console.log(window.innerWidth);  // viewport width in pixels
console.log(window.innerHeight); // viewport height in pixels
console.log(window.outerWidth);  // including browser chrome

// Global scope — var becomes window property
var globalName = "Alice";
console.log(window.globalName); // "Alice"

// let/const do NOT
let localName = "Bob";
console.log(window.localName); // undefined
```

**Output**
```
window.innerWidth   → 1440   (example viewport)
window.innerHeight  → 900
window.outerWidth   → 1440
window.globalName   → "Alice"
window.localName    → undefined
```

---

### 2. window.location — URL Management

**Theory**: `location` gives you read/write access to the current URL. You can read individual parts (pathname, search, hash), navigate to new URLs, or reload the page.

**Working Flow**
![flow-chart-2](flow-chart-2.png)

**Example**
```javascript
// Read URL parts
console.log(location.href);      // full URL
console.log(location.pathname);  // "/products"
console.log(location.search);    // "?category=shoes&sort=price"
console.log(location.hash);      // "#featured"

// Parse query parameters
const params = new URLSearchParams(location.search);
console.log(params.get("category")); // "shoes"
console.log(params.get("sort"));     // "price"

// Navigate — adds to history (back button works)
location.href = "https://example.com";
location.assign("https://example.com"); // same as above

// Navigate — replaces current entry (no back button)
location.replace("https://example.com");

// Reload the current page
location.reload();         // reload from network
location.reload(true);     // hard reload (bypass cache)
```

**Output**
```
location.pathname    → "/products"
location.search      → "?category=shoes&sort=price"
params.get("category") → "shoes"
params.get("sort")     → "price"
```

---

### 3. window.history — Navigation Control

**Theory**: `history` lets you navigate the browser's session history (the back/forward stack) and add entries without full page reloads — the foundation of single-page applications (SPAs).

**Working Flow**
![flow-chart-3](flow-chart-3.png)

**Example**
```javascript
// Read history info
console.log(history.length); // number of pages in current session

// Navigate
history.back();    // go back one page
history.forward(); // go forward one page
history.go(-2);    // go back two pages

// pushState — SPA routing
history.pushState(
  { page: "products", filter: "shoes" }, // state object
  "",                                     // title (ignored by most browsers)
  "/products?category=shoes"              // new URL
);
// → URL changes to /products?category=shoes, NO reload

// Listen for back/forward button
window.addEventListener("popstate", (event) => {
  console.log("Navigated to:", location.pathname);
  console.log("State:", event.state); // { page: "products", filter: "shoes" }
  // re-render your SPA based on the new URL
  renderPage(event.state);
});

// replaceState — update without adding history entry
history.replaceState(
  { page: "products" },
  "",
  "/products" // clean URL without params
);
```

---

### 4. window.navigator — Browser and Device Info

**Theory**: `navigator` provides information about the browser, operating system, and user's environment. Useful for feature detection, analytics, and conditional behavior.

**Working Flow**
![flow-chart-4](flow-chart-4.png)

**Example**
```javascript
// Basic info
console.log(navigator.userAgent);
// "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537..."

console.log(navigator.language);  // "en-US"
console.log(navigator.onLine);    // true or false

// Detect if online/offline
window.addEventListener("online",  () => showToast("Back online!"));
window.addEventListener("offline", () => showToast("No internet connection"));

// Geolocation
navigator.geolocation.getCurrentPosition(
  (position) => {
    const { latitude, longitude } = position.coords;
    console.log(`Location: ${latitude}, ${longitude}`);
    showMapAt(latitude, longitude);
  },
  (error) => {
    console.log("Geolocation denied:", error.message);
  }
);

// Clipboard — copy text
async function copyToClipboard(text) {
  try {
    await navigator.clipboard.writeText(text);
    showToast("Copied!");
  } catch (e) {
    console.log("Clipboard access denied");
  }
}
```

**Output**
```
navigator.language   → "en-US"
navigator.onLine     → true
geolocation success  → Location: 28.6139, 77.2090
```

---

### 5. setTimeout and setInterval — Timers

**Theory**: Timers schedule code to run in the future. `setTimeout` runs once after a delay. `setInterval` runs repeatedly. Both are macrotasks handled by the Web API — they don't block the call stack. Always store the ID returned so you can cancel them.

**Working Flow**
![flow-chart-5](flow-chart-5.png)

**Example**
```javascript
// setTimeout — run once after delay
const timerId = setTimeout(() => {
  console.log("Runs after 2 seconds");
}, 2000);

// Cancel if needed (e.g., user navigated away)
clearTimeout(timerId);

// setInterval — run every N milliseconds
let count = 0;
const intervalId = setInterval(() => {
  count++;
  console.log(`Tick ${count}`);
  if (count >= 5) {
    clearInterval(intervalId); // stop after 5 ticks
  }
}, 1000);

// Real-world: auto-save
let autoSaveTimer;
function onUserTyping() {
  clearTimeout(autoSaveTimer); // reset timer on each keystroke
  autoSaveTimer = setTimeout(() => {
    saveDocument(); // saves 2 seconds after user stops typing
  }, 2000);
}

// Real-world: countdown timer
function startCountdown(seconds) {
  let remaining = seconds;
  const display = document.querySelector("#countdown");

  const interval = setInterval(() => {
    display.textContent = remaining;
    remaining--;
    if (remaining < 0) {
      clearInterval(interval);
      display.textContent = "Time's up!";
    }
  }, 1000);
}
```

**Output** (setInterval with 5 ticks)
```
Tick 1    (after 1s)
Tick 2    (after 2s)
Tick 3    (after 3s)
Tick 4    (after 4s)
Tick 5    (after 5s)
[interval stopped]
```

---

### 6. localStorage and sessionStorage — Client-Side Storage

**Theory**: Both provide key-value storage in the browser. `localStorage` persists across tabs and browser restarts (until explicitly cleared). `sessionStorage` lasts only for the current tab session — cleared when the tab closes. Both store strings only — use `JSON.stringify`/`JSON.parse` for objects.

**Working Flow**
![flow-chart-6](flow-chart-6.png)

**Example**
```javascript
// Store user preferences
function savePreferences(prefs) {
  localStorage.setItem("theme", prefs.theme);
  localStorage.setItem("language", prefs.language);
  localStorage.setItem("user", JSON.stringify(prefs.user)); // object → string
}

// Load preferences on startup
function loadPreferences() {
  const theme = localStorage.getItem("theme") || "light"; // fallback default
  const language = localStorage.getItem("language") || "en";
  const userString = localStorage.getItem("user");
  const user = userString ? JSON.parse(userString) : null; // string → object

  return { theme, language, user };
}

// Remove on logout
function logout() {
  localStorage.removeItem("user");
  localStorage.removeItem("token");
  // or clear everything:
  localStorage.clear();
  location.href = "/login";
}

// sessionStorage — temporary data (e.g., form progress)
function saveFormProgress(step, data) {
  sessionStorage.setItem(`form_step_${step}`, JSON.stringify(data));
}

function getFormProgress(step) {
  const saved = sessionStorage.getItem(`form_step_${step}`);
  return saved ? JSON.parse(saved) : null;
}
```

---

### Real-World Examples

**Theme toggle stored in localStorage**
```javascript
const toggle = document.querySelector("#theme-toggle");
const savedTheme = localStorage.getItem("theme") || "light";
document.documentElement.setAttribute("data-theme", savedTheme);

toggle.addEventListener("click", () => {
  const current = document.documentElement.getAttribute("data-theme");
  const next = current === "light" ? "dark" : "light";
  document.documentElement.setAttribute("data-theme", next);
  localStorage.setItem("theme", next);
});
```

**Redirect with location after login**
```javascript
function redirectAfterLogin(token) {
  localStorage.setItem("auth_token", token);
  const returnTo = sessionStorage.getItem("return_url") || "/dashboard";
  sessionStorage.removeItem("return_url");
  location.href = returnTo;
}

// On protected pages — save where user was trying to go
function requireAuth() {
  if (!localStorage.getItem("auth_token")) {
    sessionStorage.setItem("return_url", location.pathname);
    location.href = "/login";
  }
}
```

---

[View Interview Questions](./interview.md)
