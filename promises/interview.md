# Promises Interview Questions

1. **What are the three states of a Promise?**
   - **Pending**: The initial state.
   - **Fulfilled**: The operation completed successfully.
   - **Rejected**: The operation failed.

2. **What is "Callback Hell" and how do Promises solve it?**
   - Callback hell occurs when you have nested callbacks within callbacks, making code unreadable and hard to maintain. Promises solve this by allowing **chaining** with `.then()`, keeping the code flat and readable.

3. **What is the difference between `Promise.all()` and `Promise.allSettled()`?**
   - `Promise.all()` fails if **any** promise in the array rejects.
   - `Promise.allSettled()` waits for all promises to finish (settle) and returns an array of objects describing the outcome of each one, regardless of success or failure.

4. **What does `Promise.race()` do?**
   - It returns a promise that fulfills or rejects as soon as **one** of the promises in the array settles (either successfully or with an error).

5. **What is the purpose of the `.finally()` method?**
   - It allows you to execute code regardless of whether the promise was fulfilled or rejected. It is commonly used for cleanup (e.g., hiding a loading spinner).

6. **What happens if a error occurs in a `.then()` block and there is no `.catch()`?**
   - The error will propagate down the chain. If it reaches the end without a `.catch()`, it becomes an "Uncaught Promise Rejection" error.

7. **How do you convert a callback-based function to a Promise-based one?**
   - This is called **Promisification**. You wrap the callback function in a `new Promise` constructor and call `resolve()` on success and `reject()` on error.
