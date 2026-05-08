# Async / Await Interview Questions

1. **What is Async/Await?**
   - It is syntactic sugar built on top of Promises. It allows you to write asynchronous code in a way that looks like synchronous, sequential code.

2. **What does the `async` keyword do?**
   - It ensures that the function always returns a Promise. If the function returns a non-promise value, it is automatically wrapped in a resolved Promise.

3. **Can you use `await` outside of an `async` function?**
   - In standard scripts, no. However, modern environments (like ES Modules) support "Top-level await" in the main body of the module.

4. **How do you handle errors in Async/Await?**
   - You use standard `try...catch` blocks. This is one of the main advantages over Promise chains, as it keeps error handling consistent with synchronous code.

5. **Does `await` block the entire main thread?**
   - **No.** `await` only pauses the execution of that specific `async` function. The JavaScript engine is free to go and do other things (like handling UI events or other requests) while the promise is pending.

6. **How do you run multiple async tasks in parallel with Async/Await?**
   - Use `Promise.all([task1(), task2()])` and `await` the result of that call. If you `await` each task individually, they will run one after another (sequentially), which is slower.

7. **What is the output of `console.log(greet())` if `greet` is an `async` function?**
   - It will print `Promise { <pending> }`. Even if the function body is synchronous, an `async` function always returns a promise.
