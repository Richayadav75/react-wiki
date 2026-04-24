# Promises Interview Questions

1. **What are the three states of a Promise?**
   - Pending, Fulfilled (Resolved), and Rejected.

2. **What is "Promise Chaining"?**
   - Returning a new promise from a `.then()` handler to perform sequential asynchronous operations.

3. **Difference between Promise.all and Promise.allSettled?**
   - `Promise.all` rejects immediately if any promise fails. `Promise.allSettled` waits for all promises to finish and returns their individual results/errors.
