- Category: JavaScript
- Difficulty: Advanced
- Related: promises, async-await

### The Event Loop — How JavaScript Handles Time

JavaScript is **single-threaded**, meaning only one thing runs at a time. Yet it handles timers, network calls, and user clicks without freezing. The **Event Loop** is the engine behind that trick — it coordinates the call stack, Web APIs, microtask queue, and macrotask queue to create the illusion of concurrency.

**Analogy**
A busy chef (JS engine) with one pair of hands (single thread). When they need bread toasted (async task), they put the bread in the toaster (Web API) and keep chopping vegetables. When the toaster dings, the chef finishes their current task, then picks up the toast. The chef never stands idle waiting — the toaster runs in the background.

---

### 1. The Four Components

**Theory**: Everything in JS async behavior flows through four parts working together.

**Working Flow**
![flow-chart](flow-chart.png)

**Explanation**
- **Call Stack**: Where synchronous code runs. Functions are pushed on entry, popped on return.
- **Web APIs**: Browser-provided environment. Handles timers, network, DOM events — outside the JS engine.
- **Microtask Queue**: High-priority queue. Drains completely before any macrotask runs.
- **Macrotask Queue**: Lower-priority. One task is picked per loop iteration.
- **Event Loop**: Monitors the stack — when it empties, it first flushes all microtasks, then picks one macrotask.

---

### 2. Execution Priority Order

**Theory**: The loop follows a strict priority. Synchronous code always runs first. Then every pending microtask runs. Then exactly one macrotask runs. Then microtasks again. This repeats.

**Working Flow**
![flow-chart-2](flow-chart-2.png)

**Example**
```javascript
console.log("1");                              // sync

setTimeout(() => console.log("4"), 0);         // macrotask

Promise.resolve().then(() => console.log("3")); // microtask

console.log("2");                              // sync
```

**Output**
```
1
2
3
4
```

**Explanation**
- `"1"` and `"2"` are synchronous — they run in order on the call stack.
- `setTimeout(fn, 0)` sends `fn` to the macrotask queue — even 0ms means "after current work".
- `Promise.resolve().then(fn)` queues `fn` in the microtask queue.
- After the stack clears (`"2"` finishes), the event loop checks: microtasks first → prints `"3"`.
- Then one macrotask runs → prints `"4"`.

---

### 3. The Classic Interview Question — Step by Step

**Theory**: Understanding this example proves you understand the event loop.

**Example**
```javascript
console.log(1);

setTimeout(() => console.log(2), 0);

Promise.resolve().then(() => console.log(3));

console.log(4);
```

**Output**
```
1
4
3
2
```

**Working Flow**
![flow-chart-3](flow-chart-3.png)

---

### 4. Multiple Microtasks — All Drain Before Macrotask

**Theory**: No matter how many `.then()` chains you have, ALL microtasks run before the next `setTimeout` fires.

**Example**
```javascript
console.log("start");

setTimeout(() => console.log("timeout"), 0);

Promise.resolve()
  .then(() => {
    console.log("promise 1");
    return "chained";
  })
  .then((val) => console.log("promise 2 →", val));

console.log("end");
```

**Output**
```
start
end
promise 1
promise 2 → chained
timeout
```

**Explanation**
Both `.then()` callbacks are microtasks. When `"promise 1"` runs and returns a value, the next `.then` is immediately queued as another microtask — before `timeout` ever runs.

---

### 5. async/await Under the Hood

**Theory**: `async/await` is syntactic sugar over Promises. Every `await` pauses the function and queues the rest as a microtask.

**Example**
```javascript
async function fetchData() {
  console.log("inside async - before await");
  await Promise.resolve();
  console.log("inside async - after await"); // runs as microtask
}

console.log("before call");
fetchData();
console.log("after call");
```

**Output**
```
before call
inside async - before await
after call
inside async - after await
```

**Working Flow**
![flow-chart-4](flow-chart-4.png)

---

### 6. Why UI Freezes — Blocking the Stack

**Theory**: Since the event loop only checks queues when the stack is empty, a long synchronous loop blocks everything — no renders, no clicks, no network responses.

**Example**
```javascript
// BAD: blocks UI for ~3 seconds
function blockingLoop() {
  const start = Date.now();
  while (Date.now() - start < 3000) { /* nothing */ }
  console.log("done blocking");
}

blockingLoop();
// The browser cannot repaint or handle clicks for 3 seconds
```

**Working Flow**
![flow-chart-5](flow-chart-5.png)

**Fix — Break work into chunks using setTimeout**
```javascript
function nonBlockingChunk(items, index = 0) {
  if (index >= items.length) return;

  // process one item
  processItem(items[index]);

  // yield to event loop, then continue
  setTimeout(() => nonBlockingChunk(items, index + 1), 0);
}
```

---

### Real-World Examples

**Debounce — uses macrotask queue to delay execution**
```javascript
function debounce(fn, delay) {
  let timer;
  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}

const onSearch = debounce((query) => {
  console.log("Searching:", query);
}, 300);

// Called rapidly as user types — only fires 300ms after they stop
```

**Priority loading — microtasks for critical, macrotask for background**
```javascript
async function loadPage() {
  // Critical data — microtask priority
  const user = await fetchUser();
  renderHeader(user);

  // Non-critical — pushed to macrotask so UI renders first
  setTimeout(() => {
    loadRecommendations();
  }, 0);
}
```

---

[View Interview Questions](./interview.md)
