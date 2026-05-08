# Event Loop Interview Questions

1. **Is JavaScript single-threaded or multi-threaded?**
   - JavaScript is single-threaded, meaning it has only one call stack and can execute one command at a time. However, the browser provides Web APIs that allow for asynchronous, non-blocking behavior.

2. **What is the difference between the Call Stack and the Task Queue?**
   - The **Call Stack** is where synchronous code is executed immediately.
   - The **Task Queue** is where asynchronous callbacks (like `setTimeout`) wait to be executed once the call stack is empty.

3. **What is the difference between a Macrotask and a Microtask?**
   - **Microtasks** (Promises, `process.nextTick`) have higher priority. The event loop will clear the entire microtask queue before moving on to the next macrotask.
   - **Macrotasks** (Timeouts, intervals, I/O) are executed one by one after the microtask queue is empty.

4. **What is the output of `setTimeout(() => console.log(1), 0); console.log(2);`?**
   - Output: `2`, then `1`. Even with a delay of `0`, the timeout callback is sent to the task queue and must wait for the main stack (`console.log(2)`) to clear.

5. **Why can't `setTimeout` guarantee an exact execution time?**
   - Because the callback must wait for the call stack to be empty. If there is a long-running synchronous task on the stack, the timeout callback will be delayed until that task finishes.

6. **What happens if you have an infinite loop of Microtasks?**
   - It will block the Macrotask queue and the UI rendering indefinitely, effectively crashing the tab. This is because the event loop won't move to the next macrotask until the microtask queue is empty.

7. **How does the Event Loop handle UI rendering?**
   - Rendering usually happens after the microtask queue is cleared and before the next macrotask is processed.
 Riverside.
 Riverside.
