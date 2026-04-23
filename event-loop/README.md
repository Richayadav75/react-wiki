- Category: Core Concepts of JavaScript
- Difficulty: Advanced
- Related: Promises, Async/Await

# Event Loop

The Event Loop is what allows JavaScript to perform non-blocking I/O operations, despite being single-threaded, by offloading operations to the system kernel whenever possible.

## How it works
1. **Call Stack**: JavaScript executes code one line at a time here.
2. **Web APIs**: When an async task starts (like `setTimeout`), it's handled by the browser.
3. **Callback Queue**: When the task finishes, its callback moves here.
4. **Microtask Queue**: Promises and MutationObservers go here. They have higher priority than the Callback Queue.
5. **The Loop**: If the Call Stack is empty, the Event Loop takes the first task from the Microtask Queue (or Callback Queue if Microtask is empty) and pushes it onto the stack.

## Practice
```javascript
console.log("1");

setTimeout(() => {
  console.log("2");
}, 0);

Promise.resolve().then(() => {
  console.log("3");
});

console.log("4");
// Expected: 1, 4, 3, 2
```

## Interview Questions
1. **What is the difference between a Task and a Microtask?**
   Microtasks (Promises) are executed immediately after the current task, before the next event loop tick. Tasks (setTimeout) go to the end of the callback queue.
2. **Is JavaScript multi-threaded?**
   No, JavaScript is single-threaded, but the environment (browser/Node) provides Web APIs that handle async tasks in the background.
