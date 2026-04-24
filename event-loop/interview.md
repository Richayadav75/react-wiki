# Event Loop Interview Questions

1. **What is the Call Stack?**
   - A LIFO (Last In, First Out) data structure that tracks the current execution context of the program.

2. **Difference between Task Queue and Microtask Queue?**
   - Microtasks (e.g., Promises, MutationObserver) have higher priority. The Event Loop processes all microtasks before moving to the next task in the Task Queue (e.g., `setTimeout`, DOM events).

3. **Is JavaScript multi-threaded?**
   - No, it's single-threaded. However, the browser environment (Web APIs) provides multi-threaded capabilities for things like timers and network requests.
