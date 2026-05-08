# Error Handling Interview Questions

---

**1. What are the three blocks of try/catch/finally and when does each run?**

- `try` — wrap code that might fail. Runs normally until an error is thrown.
- `catch(e)` — runs only when an error is thrown in `try`. Receives the error object.
- `finally` — always runs, whether or not an error occurred. Use it to release resources: hide spinners, close DB connections, reset flags.

```javascript
try {
  riskyOperation();
} catch (e) {
  console.log("Error:", e.message);
} finally {
  setLoading(false); // always runs
}
```

---

**2. Can try/catch catch errors inside setTimeout?**

No. Standard `try/catch` cannot catch errors thrown inside `setTimeout` because by the time the callback runs, the `try` block has already finished. The error escapes to the global scope.

```javascript
// BROKEN — catch never fires
try {
  setTimeout(() => { throw new Error("boom"); }, 0);
} catch (e) {
  console.log("Caught"); // never prints
}

// FIX — use async/await
async function run() {
  try {
    await new Promise((_, reject) => setTimeout(() => reject(new Error("boom")), 0));
  } catch (e) {
    console.log("Caught:", e.message); // works
  }
}
```

---

**3. What is the difference between TypeError and ReferenceError?**

- `ReferenceError` — you referenced a variable that doesn't exist in any accessible scope.
- `TypeError` — the variable exists but you're using it in a way its type doesn't support (calling a non-function, accessing property of null/undefined).

```javascript
console.log(ghost);    // ReferenceError: ghost is not defined
null.toUpperCase();    // TypeError: Cannot read properties of null
(42)();                // TypeError: 42 is not a function
```

---

**4. Why does fetch not throw on 404 or 500 errors?**

`fetch()` only rejects if the network request itself fails (no connection, DNS failure). A server responding with 404 or 500 is still a successful network request — the promise resolves. You must check `response.ok` (true for 200-299) to detect HTTP errors:

```javascript
async function get(url) {
  const res = await fetch(url);
  if (!res.ok) {
    throw new Error(`HTTP error: ${res.status}`);
  }
  return res.json();
}
```

---

**5. How do you create and use a custom Error class?**

Extend the built-in `Error` class. Call `super(message)` to set `this.message`, then set `this.name` so `instanceof` checks work correctly.

```javascript
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = "ValidationError";
    this.field = field;
  }
}

try {
  throw new ValidationError("Email required", "email");
} catch (e) {
  if (e instanceof ValidationError) {
    console.log(`${e.field}: ${e.message}`);
    // email: Email required
  }
}
```

---

**6. What is error re-throwing and why is it important?**

Re-throwing means catching an error, checking if you know how to handle it, and if not — throwing it again so a higher-level handler can deal with it. Without re-throwing, unrelated errors are silently swallowed.

```javascript
function processData(data) {
  try {
    validate(data);
  } catch (e) {
    if (e instanceof ValidationError) {
      showError(e.message); // handle it
    } else {
      throw e; // unknown error — propagate up
    }
  }
}
```

---

**7. What is the difference between `error.name`, `error.message`, and `error.stack`?**

| Property | Value | Example |
|---|---|---|
| `error.name` | Type of error | `"TypeError"` |
| `error.message` | Human-readable description | `"Cannot read properties of null"` |
| `error.stack` | Full stack trace string | `"TypeError: ...\n  at fn (file.js:10)"` |

`stack` is invaluable for debugging — it shows exactly where the error originated.

---

**8. How do you handle unhandled promise rejections globally?**

Listen for the `unhandledrejection` event on `window`. This catches any rejected promise that didn't have a `.catch()` or `try/catch` around its `await`:

```javascript
window.addEventListener("unhandledrejection", (event) => {
  console.error("Unhandled rejection:", event.reason);
  event.preventDefault(); // suppress default browser console error
  reportToErrorTracker(event.reason);
});
```

For Node.js: `process.on("unhandledRejection", (reason) => { ... })`.

---

**9. When should you use finally vs cleanup after try/catch?**

Use `finally` when the cleanup must run regardless of the outcome — including when an error is re-thrown:

```javascript
async function loadData() {
  setLoading(true);
  try {
    const data = await fetch("/api/data").then(r => r.json());
    render(data);
  } catch (e) {
    showError(e.message);
    throw e; // re-thrown — code after the block won't run
  } finally {
    setLoading(false); // always runs even after re-throw
  }
}
```

If you only put `setLoading(false)` after the catch block, it won't run when the error is re-thrown.

---

**10. What is error boundary in React vs try/catch in JavaScript?**

`try/catch` only works for imperative code (function calls, async operations). React's rendering is declarative — errors during render, lifecycle methods, or constructors cannot be caught with `try/catch`. React provides **Error Boundaries** (class components with `componentDidCatch`) that catch render-time errors and display fallback UI instead of crashing the whole tree. `try/catch` is still used for event handlers and async code inside React components.
