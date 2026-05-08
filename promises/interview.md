# Promises — Interview Questions

---

**1. What are the three states of a Promise? Can a promise change state more than once?**

A Promise has exactly three states:
- **Pending** — initial state, operation in progress
- **Fulfilled** — operation completed successfully (value available)
- **Rejected** — operation failed (reason/error available)

Once a promise moves from `pending` to either `fulfilled` or `rejected`, it is **settled** and its state is **permanent** — it can never change again.

```javascript
const p = new Promise((resolve, reject) => {
  resolve("first");
  resolve("second"); // ignored — already settled
  reject("too late");// ignored — already settled
});
p.then(v => console.log(v)); // "first"
```

---

**2. What is the difference between `.then()`, `.catch()`, and `.finally()`?**

- `.then(fn)` — runs `fn` with the resolved value when the promise fulfills
- `.catch(fn)` — runs `fn` with the rejection reason when the promise rejects
- `.finally(fn)` — runs `fn` with no argument whether it fulfilled or rejected; used for cleanup (hiding loaders, closing connections)

```javascript
fetch("/api/data")
  .then(res => res.json())              // on success
  .then(data => console.log(data))      // chain
  .catch(err => console.error(err))     // on any error in chain
  .finally(() => setLoading(false));    // always — hide spinner
```

---

**3. What is Promise chaining and why is it better than nested callbacks?**

Promise chaining returns a new promise from each `.then()`, allowing the next `.then()` to receive its result. This produces **flat, readable, top-to-bottom** code instead of deeply nested callback pyramids. Errors are caught by a single `.catch()` at the end.

```javascript
// Callback hell (before promises)
getUser(1, user => {
  getPosts(user.id, posts => {
    getComments(posts[0].id, comments => {
      // deeply nested — hard to read, error-handling a nightmare
    });
  });
});

// Promise chain (clean)
getUser(1)
  .then(user  => getPosts(user.id))
  .then(posts => getComments(posts[0].id))
  .then(comments => console.log(comments))
  .catch(err  => console.error("Any step failed:", err));
```

---

**4. What is the difference between `Promise.all` and `Promise.allSettled`?**

| | `Promise.all` | `Promise.allSettled` |
|---|---|---|
| Resolves when | ALL resolve | ALL settle (resolve or reject) |
| Rejects when | ANY rejects (fail-fast) | Never rejects |
| Result shape | `[value, value, value]` | `[{status,value}, {status,reason}, ...]` |
| Use case | Must-have-all data | Partial success is acceptable |

```javascript
Promise.all([Promise.resolve(1), Promise.reject("x"), Promise.resolve(3)])
  .catch(err => console.log("all:", err));  // "all: x" (stops at first reject)

Promise.allSettled([Promise.resolve(1), Promise.reject("x"), Promise.resolve(3)])
  .then(results => console.log(results));
// [{status:"fulfilled",value:1},{status:"rejected",reason:"x"},{status:"fulfilled",value:3}]
```

---

**5. What is the difference between `Promise.race` and `Promise.any`?**

- `Promise.race` — resolves/rejects with the **first settled** promise (first to finish, win or lose)
- `Promise.any` — resolves with the **first fulfilled** promise; ignores rejections; rejects only if ALL reject (`AggregateError`)

```javascript
const fast  = new Promise((_, rej) => setTimeout(() => rej("fast fail"), 100));
const slow  = new Promise(res => setTimeout(() => res("slow win"), 500));

Promise.race([fast, slow]).catch(e => console.log("race:", e));  // "race: fast fail"
Promise.any ([fast, slow]).then(v  => console.log("any:", v));   // "any: slow win"
```

---

**6. How do you create a Promise that resolves or rejects immediately?**

```javascript
// Immediately fulfilled
const p1 = Promise.resolve(42);
p1.then(v => console.log(v)); // 42

// Immediately rejected
const p2 = Promise.reject(new Error("oops"));
p2.catch(e => console.log(e.message)); // "oops"

// Useful in tests and fallbacks
function getConfig(useDefault) {
  if (useDefault) return Promise.resolve({ timeout: 3000 });
  return fetch("/api/config").then(r => r.json());
}
```

---

**7. What is "promisification"? Show an example.**

Promisification wraps a callback-based function into a function that returns a Promise. This makes old callback APIs compatible with modern async/await code.

```javascript
const fs = require("fs");

// Promisified version
function readFile(path) {
  return new Promise((resolve, reject) => {
    fs.readFile(path, "utf8", (err, data) => {
      if (err) reject(err);
      else     resolve(data);
    });
  });
}

readFile("./data.json")
  .then(content => console.log(content))
  .catch(err    => console.error(err.message));

// Node.js ships util.promisify for this
const { promisify } = require("util");
const readFileAsync = promisify(fs.readFile);
```

---

**8. What happens if you don't return a value inside a `.then()` callback?**

If you don't return anything, the next `.then()` receives `undefined`. If you return a plain value, it is wrapped in a resolved promise. If you return a promise, the chain waits for that promise to settle.

```javascript
Promise.resolve(1)
  .then(v => v + 1)       // returns 2
  .then(v => {
    console.log(v);        // 2
    // no return → next gets undefined
  })
  .then(v => console.log(v)); // undefined

// Returning a promise — chain waits for it
Promise.resolve("start")
  .then(v => new Promise(res => setTimeout(() => res("done"), 500)))
  .then(v => console.log(v)); // "done" (after 500ms)
```

---

**9. How do you implement a timeout for a Promise using `Promise.race`?**

```javascript
function withTimeout(promise, ms) {
  const timeout = new Promise((_, reject) =>
    setTimeout(() => reject(new Error(`Timed out after ${ms}ms`)), ms)
  );
  return Promise.race([promise, timeout]);
}

const slowFetch = new Promise(res => setTimeout(() => res("data"), 3000));

withTimeout(slowFetch, 1000)
  .then(v   => console.log("Got:", v))
  .catch(err => console.log(err.message));  // "Timed out after 1000ms"
```

---

**10. What is the difference between parallel and sequential promise execution?**

**Sequential** — await each promise in turn; total time = sum of all waits.
**Parallel** — start all promises at once with `Promise.all`; total time = slowest one.

```javascript
// Sequential (~600ms total)
async function sequential() {
  const a = await fetchA();   // 200ms — waits before starting fetchB
  const b = await fetchB();   // 200ms
  const c = await fetchC();   // 200ms
  return [a, b, c];
}

// Parallel (~200ms total — all run at the same time)
async function parallel() {
  const [a, b, c] = await Promise.all([fetchA(), fetchB(), fetchC()]);
  return [a, b, c];
}

// Common mistake — this is NOT parallel:
const pA = await fetchA();   // awaits immediately — sequential!
const pB = await fetchB();

// Correct parallel start:
const promA = fetchA();      // start both (no await yet)
const promB = fetchB();
const [a, b] = await Promise.all([promA, promB]); // now await both
```
