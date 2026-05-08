- Category: JavaScript
- Difficulty: Intermediate
- Related: async-await, event-loop, functions

### Promises — Managing Asynchronous Operations
A **Promise** is an object that represents the eventual completion (or failure) of an asynchronous operation and its resulting value. It is the backbone of modern async JavaScript — solving "callback hell" and providing clean, chainable async code.

**Analogy**
Ordering food at a restaurant. You order (start async operation). The waiter gives you a buzzer (promise) and says "I'll let you know when it's ready." You don't stand at the counter waiting — you go sit down and do other things (non-blocking). When the food is ready (resolved), your buzzer goes off and you collect it. If they run out (rejected), they tell you and you handle it (.catch).

---

### 1. The Three States of a Promise

**Theory**: A promise is always in one of three states. Once it moves from `pending` to either `fulfilled` or `rejected`, it is **settled** and can never change state again.

**Working Flow**
![flow-chart](flow-chart.png)

**Example**
```javascript
// Creating a promise
const p1 = new Promise((resolve, reject) => {
  resolve("success!");   // fulfilled immediately
});

const p2 = new Promise((resolve, reject) => {
  reject(new Error("something failed")); // rejected immediately
});

const p3 = new Promise((resolve) => {
  setTimeout(() => resolve("done after 1s"), 1000); // pending for 1s
});

p1.then(val => console.log("p1:", val));   // "p1: success!"
p2.catch(err => console.log("p2:", err.message)); // "p2: something failed"
p3.then(val => console.log("p3:", val));   // "p3: done after 1s" (after 1s)
```

**Output**
```
p1: success!                    (immediate)
p2: something failed            (immediate)
p3: done after 1s               (after 1 second)
```

---

### 2. .then(), .catch(), .finally()

**Theory**: These are the three handlers for dealing with promise results.
- `.then(onFulfilled)` — runs when the promise resolves, receives the value
- `.catch(onRejected)` — runs when the promise rejects, receives the error
- `.finally(fn)` — runs whether the promise fulfilled or rejected — great for cleanup

**Working Flow**
![flow-chart-2](flow-chart-2.png)

**Example**
```javascript
function fetchUser(id) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (id > 0) {
        resolve({ id, name: "Alice", role: "admin" });
      } else {
        reject(new Error("Invalid user ID"));
      }
    }, 500);
  });
}

// Success path
fetchUser(1)
  .then(user => {
    console.log("Got user:", user.name);  // "Got user: Alice"
    return user.role;                     // pass to next .then
  })
  .then(role => {
    console.log("Role:", role);           // "Role: admin"
  })
  .catch(err => {
    console.log("Error:", err.message);
  })
  .finally(() => {
    console.log("Request complete");      // always runs
  });

// Error path
fetchUser(-1)
  .then(user  => console.log(user.name))
  .catch(err  => console.log("Error:", err.message)) // "Error: Invalid user ID"
  .finally(() => console.log("Request complete"));
```

**Output**
```
Got user: Alice
Role: admin
Request complete

Error: Invalid user ID
Request complete
```

---

### 3. Promise Chaining vs Callback Hell

**Theory**: Before Promises, async operations were nested inside callbacks, creating deeply indented "pyramid of doom" code. Promise chaining flattens this into a readable top-to-bottom sequence.

**Working Flow**
![flow-chart-3](flow-chart-3.png)

**Example**
```javascript
// Simulated API functions
const getUser    = id  => new Promise(res => setTimeout(() => res({ id, name: "Alice" }), 200));
const getPosts   = usr => new Promise(res => setTimeout(() => res([{ id: 1, title: "Hello JS" }]), 200));
const getComment = pst => new Promise(res => setTimeout(() => res({ text: "Great post!" }), 200));

// Clean promise chain
getUser(1)
  .then(user => {
    console.log("User:", user.name);
    return getPosts(user);     // return next promise
  })
  .then(posts => {
    console.log("Post:", posts[0].title);
    return getComment(posts[0]); // return next promise
  })
  .then(comment => {
    console.log("Comment:", comment.text);
  })
  .catch(err => {
    console.log("Chain failed:", err.message);
  });
```

**Output**
```
User: Alice
Post: Hello JS
Comment: Great post!
```

**Explanation**: Each `.then` receives the return value of the previous one. If any promise in the chain rejects, execution jumps to `.catch` — no need to handle errors at every level.

---

### 4. Promise.all — Wait for All

**Theory**: `Promise.all([...])` takes an array of promises and returns a new promise that resolves when **all** input promises resolve. If **any one** rejects, the whole thing rejects immediately (fail-fast).

**Working Flow**
![flow-chart-4](flow-chart-4.png)

**Example**
```javascript
const fetchName  = () => new Promise(res => setTimeout(() => res("Alice"), 300));
const fetchAge   = () => new Promise(res => setTimeout(() => res(30),      200));
const fetchCity  = () => new Promise(res => setTimeout(() => res("Mumbai"), 100));

// All run in PARALLEL — total time ≈ 300ms (slowest), not 600ms
Promise.all([fetchName(), fetchAge(), fetchCity()])
  .then(([name, age, city]) => {
    console.log(`${name}, age ${age}, from ${city}`);
  })
  .catch(err => console.log("One failed:", err.message));

// If one rejects:
const p1 = Promise.resolve("ok");
const p2 = Promise.reject(new Error("p2 failed"));
const p3 = Promise.resolve("also ok");

Promise.all([p1, p2, p3])
  .then(vals => console.log(vals))
  .catch(err => console.log("Caught:", err.message)); // "Caught: p2 failed"
```

**Output**
```
Alice, age 30, from Mumbai   (after ~300ms)
Caught: p2 failed
```

---

### 5. Promise.allSettled — Wait for All, Never Fail

**Theory**: `Promise.allSettled([...])` waits for ALL promises to settle (resolve or reject), and always resolves with an array of result objects — one per promise — each with a `status` of `"fulfilled"` or `"rejected"`.

**Working Flow**
![flow-chart-5](flow-chart-5.png)

**Example**
```javascript
const apis = [
  Promise.resolve({ data: "users list" }),
  Promise.reject(new Error("auth failed")),
  Promise.resolve({ data: "products list" }),
];

Promise.allSettled(apis).then(results => {
  results.forEach((result, i) => {
    if (result.status === "fulfilled") {
      console.log(`API ${i+1} OK:`, result.value.data);
    } else {
      console.log(`API ${i+1} FAILED:`, result.reason.message);
    }
  });
});
```

**Output**
```
API 1 OK: users list
API 2 FAILED: auth failed
API 3 OK: products list
```

---

### 6. Promise.race & Promise.any

**Theory**:
- `Promise.race([...])` — resolves or rejects with the **first** settled promise (whichever finishes first, win or lose)
- `Promise.any([...])` — resolves with the **first fulfilled** promise; only rejects if **all** reject (AggregateError)

**Working Flow**
![flow-chart-6](flow-chart-6.png)

**Example**
```javascript
const fast = new Promise(res => setTimeout(() => res("fast!"),  100));
const slow = new Promise(res => setTimeout(() => res("slow!"), 2000));

// race — first to settle wins
Promise.race([slow, fast]).then(v => console.log("Race:", v));
// Race: fast!  (fast resolved first)

// Timeout pattern using race
function withTimeout(promise, ms) {
  const timeout = new Promise((_, rej) =>
    setTimeout(() => rej(new Error(`Timed out after ${ms}ms`)), ms)
  );
  return Promise.race([promise, timeout]);
}

withTimeout(slow, 500)
  .then(v => console.log(v))
  .catch(e => console.log(e.message)); // "Timed out after 500ms"

// any — first SUCCESS wins
const p1 = Promise.reject(new Error("fail 1"));
const p2 = new Promise(res => setTimeout(() => res("second wins!"), 200));
const p3 = Promise.reject(new Error("fail 3"));

Promise.any([p1, p2, p3]).then(v => console.log("Any:", v));
// Any: second wins!
```

**Output**
```
Race: fast!
Timed out after 500ms
Any: second wins!
```

---

### Real-World — Sequential vs Parallel API Calls

```javascript
// Sequential — each waits for the previous (total time = sum of all)
async function sequential() {
  const user    = await fetch("/api/user/1").then(r => r.json());     // 300ms
  const posts   = await fetch(`/api/posts?user=${user.id}`).then(r => r.json()); // 200ms
  const profile = await fetch(`/api/profile/${user.id}`).then(r => r.json()); // 150ms
  // Total: ~650ms
  return { user, posts, profile };
}

// Parallel — all run at the same time (total time = slowest)
async function parallel() {
  const [user, trending, ads] = await Promise.all([
    fetch("/api/user/1").then(r => r.json()),      // 300ms ─┐
    fetch("/api/trending").then(r => r.json()),    // 200ms  │ all at once
    fetch("/api/ads").then(r => r.json()),         // 100ms ─┘
  ]);
  // Total: ~300ms (slowest one)
  return { user, trending, ads };
}

// Promisification — wrap callback API in a promise
function readFile(path) {
  return new Promise((resolve, reject) => {
    fs.readFile(path, "utf8", (err, data) => {
      if (err) reject(err);
      else     resolve(data);
    });
  });
}

readFile("./config.json")
  .then(data => console.log("File:", data))
  .catch(err => console.log("Error:", err.message));
```

---

[View Interview Questions](./interview.md)
