# Async/Await Interview Questions

1. **How is async/await related to Promises?**
   - It's syntactic sugar built on top of Promises. An `async` function always returns a Promise.

2. **How do you handle errors in async/await?**
   - Using `try...catch` blocks around the `await` expressions.

3. **What is the benefit of async/await over then/catch?**
   - It makes asynchronous code look and behave more like synchronous code, improving readability and error handling.
