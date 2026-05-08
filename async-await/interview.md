# Async/Await — Interview Questions

---

**1. What does the `async` keyword do to a function?**

It makes the function always return a **Promise**, regardless of what it returns internally. If it returns a plain value, that value is wrapped in `Promise.resolve()`. If it throws, the error is wrapped in `Promise.reject()`.

```javascript
async function greet() { return "Hello"; }
async function fail()  { throw new Error("oops"); }

greet().then(v => console.log(v));          // "Hello"
fail().catch(e => console.log(e.message)); // "oops"

// Even this returns a Promise:
console.log(greet()); // Promise { "Hello" }
```

---

**2. What does `await` do, and where can it be used?**

`await` pauses the execution of the **current async function** until the awaited Promise settles. The JS engine is not blocked — it can process other tasks while waiting. `await` can only be used:
1. Inside an `async` function
2. At the top level of an ES module (top-level await)

```javascript
async function example() {
  console.log("before");
  const result = await new Promise(res => setTimeout(() => res("done"), 500));
  console.log("after:", result);  // runs after 500ms
}
console.log("start");
example();
console.log("continues");
// Output: start → before → continues → after: done
```

---

**3. Does `await` block the entire JavaScript thread?**

No. `await` only pauses the **current async function**. The JavaScript engine remains free to handle other events, timers, and tasks while the promise is pending. This is non-blocking I/O.

```javascript
async function slowOp() {
  await new Promise(res => setTimeout(res, 1000)); // pauses THIS function
  return "slow done";
}

slowOp().then(v => console.log(v));
console.log("I run immediately"); // runs before "slow done"
// JS engine handles other tasks during the 1000ms wait
```

---

**4. How do you handle errors in async/await?**

Use `try/catch` blocks. Any awaited promise that rejects throws an error inside the function — `catch` intercepts it. Use `finally` for cleanup that must always run.

```javascript
async function loadData(url) {
  try {
    const res = await fetch(url);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return await res.json();
  } catch (err) {
    console.error("Failed:", err.message);
    return null;   // return fallback
  } finally {
    console.log("Request complete"); // always runs
  }
}
```

---

**5. What is the difference between sequential and parallel async execution?**

- **Sequential** — each `await` runs after the previous finishes; total time = sum
- **Parallel** — start all promises at once with `Promise.all`, then await the group; total time = slowest one

```javascript
// Sequential — 600ms total
async function slow() {
  const a = await fetch("/api/a"); // 200ms — B won't start until A finishes
  const b = await fetch("/api/b"); // 200ms
  const c = await fetch("/api/c"); // 200ms
}

// Parallel — ~200ms total
async function fast() {
  const [a, b, c] = await Promise.all([
    fetch("/api/a"),  // all three start simultaneously
    fetch("/api/b"),
    fetch("/api/c"),
  ]);
}
```

---

**6. What is a common mistake that makes `Promise.all` run sequentially instead of in parallel?**

Awaiting each promise before passing it to `Promise.all`. By the time you call `Promise.all`, the first promise has already settled — they ran one by one.

```javascript
// WRONG — sequential! Each await runs before the next line
const p1 = await fetchA();   // waits 200ms
const p2 = await fetchB();   // then waits 200ms
await Promise.all([p1, p2]); // they're already done — no parallel benefit

// CORRECT — start all, then await together
const promA = fetchA();      // starts immediately (no await)
const promB = fetchB();      // starts immediately (no await)
const [a, b] = await Promise.all([promA, promB]); // wait for both
```

---

**7. What is an async IIFE and when would you use it?**

An **Immediately Invoked Function Expression** with `async` lets you use `await` in environments that don't support top-level await (like CommonJS scripts).

```javascript
// Regular script — can't use top-level await
// await fetch(...)  // SyntaxError!

// Async IIFE — workaround
(async () => {
  const res  = await fetch("/api/data");
  const data = await res.json();
  console.log(data);
})();

// Or with error handling
(async () => {
  try {
    const data = await loadConfig();
    initialize(data);
  } catch (err) {
    console.error("Startup failed:", err);
  }
})();
```

---

**8. What is top-level await and when is it available?**

Top-level `await` allows using `await` directly in the body of an ES module without wrapping it in an `async` function. The entire module is paused until the awaited promise settles. It is only available in **ES modules** (files with `type="module"` or `.mjs`).

```javascript
// config.mjs — valid ES module
const res    = await fetch("/api/config");
const config = await res.json();

export const API_KEY = config.apiKey;

// Any module that imports from config.mjs
// waits for it to fully load before executing
import { API_KEY } from "./config.mjs";
console.log(API_KEY); // guaranteed to be available
```

---

**9. How does error propagation work with async/await?**

If an async function doesn't catch its own error, the rejected promise propagates up the call stack. The first caller with a `try/catch` or `.catch()` will intercept it.

```javascript
async function a() { throw new Error("deep"); }
async function b() { return await a(); }      // propagates
async function c() { return await b(); }      // propagates

// Catch at the top
c().catch(err => console.log("Caught:", err.message)); // "Caught: deep"

// Or with try/catch in the final async function
async function main() {
  try {
    await c();
  } catch (err) {
    console.log("Main caught:", err.message); // "Main caught: deep"
  }
}
```

---

**10. How do you retry a failed async operation with async/await?**

Use a loop with try/catch to attempt the operation multiple times before giving up.

```javascript
async function fetchWithRetry(url, maxRetries = 3) {
  let lastError;
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const res = await fetch(url);
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      return await res.json();
    } catch (err) {
      lastError = err;
      console.log(`Attempt ${attempt} failed: ${err.message}`);
      if (attempt < maxRetries) {
        await new Promise(res => setTimeout(res, 1000 * attempt)); // backoff
      }
    }
  }
  throw new Error(`All ${maxRetries} attempts failed: ${lastError.message}`);
}

fetchWithRetry("/api/data")
  .then(data => console.log("Success:", data))
  .catch(err  => console.log("Final error:", err.message));
// Attempt 1 failed: ...
// Attempt 2 failed: ...
// Attempt 3 failed: ...
// Final error: All 3 attempts failed: ...
```
