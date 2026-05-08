# Error Handling Interview Questions

1. **What are the three parts of a `try...catch` statement?**
   - **try**: The block where you put code that might fail.
   - **catch**: The block that handles the error if one occurs.
   - **finally**: The block that runs regardless of the outcome (success or failure).

2. **Can `try...catch` catch errors in asynchronous code?**
   - Only if you use `async/await`. If you use standard callbacks (like `setTimeout`) or raw Promise chains without `await`, the `try...catch` block will finish executing before the error actually occurs, and the error will go uncaught.

3. **What is the difference between a `TypeError` and a `ReferenceError`?**
   - **ReferenceError**: Occurs when you try to access a variable that has not been declared or is out of scope.
   - **TypeError**: Occurs when a variable exists, but you try to perform an operation on it that its type doesn't support (e.g., calling a number as a function: `(5)()`).

4. **What does the `throw` keyword do?**
   - It allows you to manually create an error and stop the execution of the current function. You can throw strings, numbers, or (best practice) instances of the `Error` class.

5. **Why should you use `finally`?**
   - It is used for cleanup actions that must happen no matter what, such as closing a file stream, hiding a loading spinner, or clearing a timer.

6. **What is an "Error Object"?**
   - When an error occurs, JavaScript creates an object containing information about the failure. The two most important properties are `.name` (e.g., "ReferenceError") and `.message` (a human-readable description).

7. **How do you create a custom error?**
   - By creating a class that extends the built-in `Error` class. This allows you to use `instanceof` to check for specific error types in your catch blocks.
