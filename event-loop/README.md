- Category: Core Concepts
- Difficulty: Advanced
- Related: promises

### What is the Event Loop?
The Event Loop is the secret behind JavaScript's asynchronous behavior. It allows JS (which is single-threaded) to perform non-blocking operations.

---

### 1. Concurrency Model
**Theory**: JS executes tasks in the Call Stack. Async tasks (like `setTimeout`) are sent to Web APIs. When finished, they wait in the Task Queue until the Stack is empty.

**Working Flow**
```text
[ Call Stack ] --> ( Async Task? ) --> [ Web APIs ] --> [ Task Queue ]
      ^                                                      |
      |-------------------( Event Loop )---------------------|
```

**Key Features**:
- **Call Stack**: Where your code actually runs.
- **Task Queue**: Where callbacks wait their turn.
- **Microtask Queue**: High priority queue (for Promises).

**Example**:
```javascript
console.log("Start");

setTimeout(() => {
  console.log("Timeout (Task Queue)");
}, 0);

Promise.resolve().then(() => console.log("Promise (Microtask Queue)"));

console.log("End");
// Output order: Start -> End -> Promise -> Timeout
```
