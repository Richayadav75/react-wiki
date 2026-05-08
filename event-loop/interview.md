# Event Loop Interview Questions

---

**1. What is the Event Loop and why does JavaScript need it?**

JavaScript is single-threaded — it can only execute one operation at a time on the call stack. The Event Loop is the mechanism that allows JS to be non-blocking. When async tasks (like `setTimeout` or `fetch`) complete, their callbacks wait in queues. The event loop continuously checks: if the call stack is empty, it moves queued callbacks onto the stack for execution. Without it, any I/O operation would freeze the entire page.

---

**2. What is the output of this code and why?**

```javascript
console.log(1);
setTimeout(() => console.log(2), 0);
Promise.resolve().then(() => console.log(3));
console.log(4);
```

**Answer:**
```
1
4
3
2
```

`1` and `4` are synchronous. After the stack clears, the microtask queue (Promise) drains first → `3`. Then the macrotask (setTimeout) runs → `2`. Even with `0ms` delay, `setTimeout` is a macrotask and always runs after all microtasks.

---

**3. What is the difference between a Microtask and a Macrotask?**

| | Microtask | Macrotask |
|---|---|---|
| Examples | `Promise.then`, `queueMicrotask`, `MutationObserver` | `setTimeout`, `setInterval`, `setImmediate`, I/O callbacks |
| Priority | HIGH — runs before any macrotask | LOW — one per event loop iteration |
| When drained | Entire queue drains before next macrotask | One task per loop tick |

The key rule: after each macrotask, the entire microtask queue is drained before the next macrotask runs.

---

**4. What is the output of this code?**

```javascript
console.log("A");

setTimeout(() => console.log("B"), 0);

Promise.resolve()
  .then(() => console.log("C"))
  .then(() => console.log("D"));

console.log("E");
```

**Answer:**
```
A
E
C
D
B
```

`A` and `E` are sync. `C` is a microtask, and returning from it queues `D` as another microtask — both run before `B` (macrotask).

---

**5. What does `async/await` have to do with the event loop?**

`async/await` is built on Promises. When you `await` something, the function pauses and yields control back to the event loop. The code after `await` is scheduled as a microtask once the awaited promise resolves.

```javascript
async function run() {
  console.log("A");
  await Promise.resolve();
  console.log("B"); // queued as microtask
}

console.log("X");
run();
console.log("Y");
// Output: X, A, Y, B
```

---

**6. Why does a long `while` loop freeze the browser?**

Because the call stack never empties. The event loop can only process queued callbacks when the stack is empty. A blocking loop keeps the stack occupied, preventing browser repaints, click handlers, setTimeout callbacks, and network response processing.

Fix: break work into chunks using `setTimeout(fn, 0)` to yield between iterations.

---

**7. What is the output of this nested Promise/setTimeout?**

```javascript
setTimeout(() => {
  console.log("timeout 1");
  Promise.resolve().then(() => console.log("promise inside timeout"));
}, 0);

setTimeout(() => {
  console.log("timeout 2");
}, 0);
```

**Answer:**
```
timeout 1
promise inside timeout
timeout 2
```

After `timeout 1` runs (macrotask), the microtask queue is drained (`promise inside timeout`) before the next macrotask (`timeout 2`) runs.

---

**8. What is `queueMicrotask()` and when would you use it?**

`queueMicrotask(fn)` explicitly schedules a function in the microtask queue — same priority as `Promise.then`. Use it when you need something to run after the current synchronous code but before any macrotasks, without the overhead of creating a Promise.

```javascript
console.log("sync");
queueMicrotask(() => console.log("microtask"));
setTimeout(() => console.log("macrotask"), 0);
console.log("sync end");
// Output: sync, sync end, microtask, macrotask
```

---

**9. How does the Event Loop relate to rendering in the browser?**

The browser rendering pipeline (recalculate styles, layout, paint) runs between macrotasks — after the microtask queue is empty. Heavy macrotask work can delay rendering by an entire frame (16ms at 60fps). `requestAnimationFrame` is scheduled just before the browser paints — ideal for animation work that needs to be visually smooth.

---

**10. What is the difference between `setTimeout(fn, 0)` and `Promise.resolve().then(fn)`?**

Both schedule `fn` for later execution, but:
- `Promise.resolve().then(fn)` is a microtask — runs before any macrotask, including before the next `setTimeout`
- `setTimeout(fn, 0)` is a macrotask — runs after all microtasks are drained

Use Promise/microtask when you need guaranteed-before-next-render priority. Use setTimeout when you want to yield to the browser to repaint first.
