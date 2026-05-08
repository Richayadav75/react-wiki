- Category: JavaScript
- Difficulty: Intermediate
- Related: promises, event-loop, es6-features

### Async/Await — Writing Async Code That Reads Like Sync
`async`/`await` is syntactic sugar over Promises, introduced in ES2017. It lets you write asynchronous code in a top-to-bottom, step-by-step style — just like synchronous code — making it dramatically easier to read, write, and debug.

**Analogy**
Cooking a recipe with a single chef. The chef follows the steps one by one: "boil water, **wait** for it to boil, add pasta, **wait** 8 minutes, drain." The `await` keyword tells the chef to pause and wait before moving to the next step. Meanwhile the kitchen (JS engine) is still running — other orders (tasks) can be processed while waiting.

---

### 1. async Function — Always Returns a Promise

**Theory**: Any function prefixed with `async` automatically returns a Promise. If the function returns a value, it is automatically wrapped in `Promise.resolve(value)`. If the function throws, it is wrapped in `Promise.reject(error)`.

**Working Flow**
![flow-chart](flow-chart.png)

**Example**
```javascript
// async always returns a promise
async function add(a, b) {
  return a + b;         // auto-wrapped in Promise.resolve(a+b)
}

async function fail() {
  throw new Error("broken");  // auto-wrapped in Promise.reject(error)
}

add(2, 3).then(v => console.log(v));          // 5
fail().catch(e => console.log(e.message));    // "broken"

// Without return — promise resolves with undefined
async function noReturn() {
  console.log("side effect");
}
noReturn().then(v => console.log("got:", v));
// "side effect"
// "got: undefined"

// Explicit Promise.resolve — same result
async function wrapped() { return Promise.resolve(42); }
wrapped().then(v => console.log(v)); // 42
```

**Output**
```
add(2,3)       → 5
fail()         → "broken"
noReturn()     → "side effect", then undefined
wrapped()      → 42
```

---

### 2. await — Pause Until Promise Settles

**Theory**: `await` can only be used inside an `async` function (or at the top level of a module). It pauses execution of the **current function** until the promise settles, then resumes with the resolved value. The JS engine is not blocked — other tasks continue executing while waiting.

**Working Flow**
![flow-chart-2](flow-chart-2.png)

**Example**
```javascript
function delay(ms, value) {
  return new Promise(resolve => setTimeout(() => resolve(value), ms));
}

async function sequence() {
  console.log("Start");

  const a = await delay(300, "first");   // waits 300ms
  console.log("Got:", a);

  const b = await delay(200, "second");  // waits 200ms
  console.log("Got:", b);

  console.log("Done");
  return [a, b];
}

console.log("Before");
sequence().then(arr => console.log("Result:", arr));
console.log("After");
```

**Output**
```
Before
Start
After          ← JS continues while sequence() awaits
Got: first     (after 300ms)
Got: second    (after 200ms more)
Done
Result: ["first","second"]
```

---

### 3. try/catch for Error Handling

**Theory**: With async/await, you use standard `try/catch` blocks for error handling — no more `.catch()` chains. Any awaited promise that rejects throws an error inside the async function, which the `try/catch` catches naturally.

**Working Flow**
![flow-chart-3](flow-chart-3.png)

**Example**
```javascript
function fetchUser(id) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (id > 0) resolve({ id, name: "Alice" });
      else        reject(new Error("User not found"));
    }, 300);
  });
}

async function loadUser(id) {
  try {
    const user = await fetchUser(id);
    console.log("User:", user.name);
    return user;
  } catch (err) {
    console.log("Error:", err.message);
    return null;
  } finally {
    console.log("Request finished");  // always runs
  }
}

await loadUser(1);   // in async context
// User: Alice
// Request finished

await loadUser(-1);
// Error: User not found
// Request finished

// Error propagation — if no try/catch, error propagates to caller
async function noHandler(id) {
  const user = await fetchUser(id);  // throws if rejected
  return user;
}

noHandler(-1).catch(e => console.log("Caller caught:", e.message));
// Caller caught: User not found
```

**Output**
```
loadUser(1):   User: Alice / Request finished
loadUser(-1):  Error: User not found / Request finished
noHandler(-1): Caller caught: User not found
```

---

### 4. Sequential vs Parallel Execution

**Theory**: Naively awaiting independent promises one after another runs them sequentially — wasting time. Use `Promise.all` to start all promises simultaneously and await them together. The total time drops from the **sum** of all waits to the **maximum** single wait.

**Working Flow**
![flow-chart-4](flow-chart-4.png)

**Example**
```javascript
function api(name, ms) {
  return new Promise(res => setTimeout(() => res(`${name} done`), ms));
}

// Sequential — slow (total ~600ms)
async function sequential() {
  console.time("sequential");
  const a = await api("A", 200);   // waits 200ms, then starts B
  const b = await api("B", 200);   // waits 200ms, then starts C
  const c = await api("C", 200);   // waits 200ms
  console.timeEnd("sequential");   // ~600ms
  return [a, b, c];
}

// Parallel — fast (total ~200ms)
async function parallel() {
  console.time("parallel");
  const [a, b, c] = await Promise.all([
    api("A", 200),   // all three start simultaneously
    api("B", 200),
    api("C", 200),
  ]);
  console.timeEnd("parallel");    // ~200ms
  return [a, b, c];
}

// Mixed — sequential when B depends on A, parallel when independent
async function mixed() {
  const user  = await api("fetchUser",  300); // must have user first
  const [posts, profile] = await Promise.all([
    api("fetchPosts",  200),   // these two are independent
    api("fetchProfile",150),   // run them in parallel
  ]);
  return { user, posts, profile };
}
```

**Output**
```
sequential: ~600ms
parallel:   ~200ms
mixed:      ~500ms (300 + max(200,150))
```

---

### 5. Async IIFE & Error Propagation

**Theory**: An **Immediately Invoked Function Expression (IIFE)** with `async` lets you use `await` at the top level of a script (before modules/top-level await support). Error propagation: if an async function doesn't catch its errors, the rejection propagates up to the caller.

**Working Flow**
![flow-chart-5](flow-chart-5.png)

**Example**
```javascript
// Async IIFE — use await without wrapping in a named function
(async () => {
  try {
    const res = await fetch("https://jsonplaceholder.typicode.com/users/1");
    const user = await res.json();
    console.log("Name:", user.name);
  } catch (err) {
    console.log("IIFE error:", err.message);
  }
})();

// Error propagation
async function level3() {
  throw new Error("deep error");
}

async function level2() {
  return await level3();   // propagates up
}

async function level1() {
  return await level2();   // propagates up
}

level1().catch(err => console.log("Top level caught:", err.message));
// "Top level caught: deep error"

// Unhandled async rejection (bad practice)
async function dangerous() {
  throw new Error("unhandled");
}
dangerous(); // no .catch() — "UnhandledPromiseRejectionWarning"
```

**Output**
```
Name: Leanne Graham
Top level caught: deep error
UnhandledPromiseRejectionWarning (dangerous())
```

---

### 6. Top-Level Await in Modules

**Theory**: In ES modules (`type="module"` in HTML or `.mjs` files), you can use `await` directly at the top level without wrapping it in an `async` function. The module is treated as an async function — it won't finish loading until all top-level awaits complete.

**Working Flow**
![flow-chart-6](flow-chart-6.png)

**Example**
```javascript
// config.mjs — top-level await
const res    = await fetch("https://jsonplaceholder.typicode.com/users/1");
const config = await res.json();

export const userName = config.name;
export const userId   = config.id;

// main.mjs — waits for config.mjs to fully load before running
import { userName, userId } from "./config.mjs";
console.log(`User: ${userName} (ID: ${userId})`);
// "User: Leanne Graham (ID: 1)"

// Conditional dynamic import with top-level await
const module = await import(
  process.env.MODE === "dev" ? "./dev-api.mjs" : "./prod-api.mjs"
);
```

---

### Real-World Examples

**Sequential — Fetch user then their posts**
```javascript
async function getUserWithPosts(userId) {
  try {
    // Must get user first — post endpoint needs user.id
    const userRes = await fetch(`https://jsonplaceholder.typicode.com/users/${userId}`);
    if (!userRes.ok) throw new Error(`User fetch failed: ${userRes.status}`);
    const user = await userRes.json();

    // Now fetch posts using user data
    const postsRes = await fetch(`https://jsonplaceholder.typicode.com/posts?userId=${user.id}`);
    const posts = await postsRes.json();

    return {
      user:  { id: user.id, name: user.name, email: user.email },
      posts: posts.slice(0, 3).map(p => ({ id: p.id, title: p.title })),
    };
  } catch (err) {
    console.error("Failed:", err.message);
    throw err;   // re-throw so caller can handle
  }
}

const result = await getUserWithPosts(1);
console.log(result.user.name);     // "Leanne Graham"
console.log(result.posts.length);  // 3
```

**Parallel — Fetch multiple APIs at once**
```javascript
async function getDashboardData(userId) {
  const [user, posts, todos] = await Promise.all([
    fetch(`https://jsonplaceholder.typicode.com/users/${userId}`).then(r => r.json()),
    fetch(`https://jsonplaceholder.typicode.com/posts?userId=${userId}`).then(r => r.json()),
    fetch(`https://jsonplaceholder.typicode.com/todos?userId=${userId}`).then(r => r.json()),
  ]);

  return {
    user:      { name: user.name, email: user.email },
    postCount: posts.length,
    todoCount: todos.length,
    done:      todos.filter(t => t.completed).length,
  };
}

const dashboard = await getDashboardData(1);
console.log(`${dashboard.user.name} has ${dashboard.postCount} posts`);
// "Leanne Graham has 10 posts"
```

**Output**
```
getUserWithPosts(1):
  user.name      → "Leanne Graham"
  posts.length   → 3

getDashboardData(1):
  "Leanne Graham has 10 posts"
  todoCount      → 20
  done           → 11
```

---

[View Interview Questions](./interview.md)
