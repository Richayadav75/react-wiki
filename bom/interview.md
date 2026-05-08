# BOM Interview Questions

---

**1. What is the window object and why is it special?**

`window` is the global object in browser JavaScript. Every global variable (`var x`), every built-in function (`setTimeout`, `alert`, `fetch`), and every BOM component (`location`, `navigator`, `history`) is a property of `window`. You can omit `window.` — writing `setTimeout` and `window.setTimeout` are identical. `let` and `const` declarations do NOT become window properties.

```javascript
var age = 30;
console.log(window.age); // 30

let name = "Alice";
console.log(window.name); // undefined
```

---

**2. What is the difference between location.assign() and location.replace()?**

- `location.assign(url)` — navigates to the URL and adds the new page to history. User can press Back to return.
- `location.replace(url)` — navigates to the URL and replaces the current entry in history. User cannot go back.
- `location.href = url` — same behavior as `assign()`.

Use `replace()` for redirects after login/logout where going "back" would be confusing or insecure.

---

**3. What is the difference between localStorage and sessionStorage?**

| | localStorage | sessionStorage |
|---|---|---|
| Persistence | Survives tab close, browser restart | Cleared when tab closes |
| Scope | Shared across all tabs of same origin | Only accessible in the current tab |
| Capacity | ~5-10MB | ~5MB |
| Use case | User preferences, auth token | Multi-step form data, temp state |

Both use the same API: `setItem`, `getItem`, `removeItem`, `clear`.

---

**4. Why must you use JSON.stringify when storing objects in localStorage?**

`localStorage` only stores strings. Storing an object directly calls `.toString()` on it, resulting in `"[object Object]"`. Use `JSON.stringify` to convert to a JSON string, and `JSON.parse` to convert back.

```javascript
// BROKEN
localStorage.setItem("user", { name: "Alice" });
localStorage.getItem("user"); // "[object Object]"

// CORRECT
localStorage.setItem("user", JSON.stringify({ name: "Alice" }));
const user = JSON.parse(localStorage.getItem("user")); // { name: "Alice" }
```

---

**5. How does history.pushState enable single-page application routing?**

`pushState(state, title, url)` changes the browser's URL and adds an entry to the session history — without triggering a page reload. The JS app updates what it displays based on the new URL. Pressing Back fires the `popstate` event, which the SPA listens to and re-renders the appropriate view.

```javascript
history.pushState({ page: "about" }, "", "/about");
// URL → /about, no reload

window.addEventListener("popstate", (e) => {
  renderPage(e.state?.page || "home");
});
```

---

**6. What is the difference between setTimeout and setInterval?**

- `setTimeout(fn, ms)` — runs `fn` exactly once after `ms` milliseconds.
- `setInterval(fn, ms)` — runs `fn` repeatedly every `ms` milliseconds.

Both return an ID. Use `clearTimeout(id)` or `clearInterval(id)` to cancel. Always cancel intervals when they're no longer needed (e.g., in React's `useEffect` cleanup) to prevent memory leaks.

```javascript
// setTimeout — once
const id = setTimeout(() => console.log("once"), 1000);
clearTimeout(id); // cancel if needed

// setInterval — repeat
let count = 0;
const intervalId = setInterval(() => {
  console.log(++count);
  if (count === 3) clearInterval(intervalId);
}, 500);
// Output: 1, 2, 3
```

---

**7. What does navigator.onLine tell you and what are its limitations?**

`navigator.onLine` returns `true` if the browser believes it has a network connection, `false` if definitely offline. However:
- `true` does not guarantee internet access — you could be connected to a LAN with no internet.
- It can lag behind reality slightly.

For reliable connectivity detection, combine with a `fetch` ping or listen to the `online`/`offline` events:

```javascript
window.addEventListener("offline", () => showBanner("You are offline"));
window.addEventListener("online",  () => hideBanner());
```

---

**8. What is the difference between window.innerWidth and screen.width?**

- `window.innerWidth` — the viewport width (the visible area for content), in CSS pixels. Changes with browser zoom and responsive design.
- `screen.width` — the physical width of the user's display in pixels. Never changes regardless of browser size.

For responsive JS logic, use `window.innerWidth`. For analytics about screen size, use `screen.width`.

---

**9. How do you prevent a setInterval from causing memory leaks in React?**

Return a cleanup function from `useEffect` that calls `clearInterval`. Without it, the interval keeps running after the component unmounts:

```javascript
useEffect(() => {
  const id = setInterval(() => {
    setCount(c => c + 1);
  }, 1000);

  return () => clearInterval(id); // cleanup on unmount
}, []);
```

---

**10. How would you implement a "return to where you were" redirect after login?**

Save the intended URL to `sessionStorage` before redirecting to login, then read and clear it after successful login:

```javascript
// On protected page — save target before redirecting
function requireAuth() {
  if (!getToken()) {
    sessionStorage.setItem("returnUrl", location.pathname + location.search);
    location.href = "/login";
  }
}

// After login success
function onLoginSuccess(token) {
  saveToken(token);
  const returnUrl = sessionStorage.getItem("returnUrl") || "/dashboard";
  sessionStorage.removeItem("returnUrl");
  location.href = returnUrl;
}
```
