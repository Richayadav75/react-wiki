- Category: JavaScript
- Difficulty: Intermediate
- Related: Promises, Event Loop

# Async/Await

Async/Await is a syntax sugar built on top of Promises. It makes asynchronous code look and behave more like synchronous code.

## How it works
- `async` keyword: Declares that a function returns a promise.
- `await` keyword: Pauses the execution of the async function until the promise is settled.

## Example
```javascript
async function fetchData() {
  try {
    const response = await fetch('https://api.github.com');
    const data = await response.json();
    console.log(data);
  } catch (error) {
    console.error('Error fetching data:', error);
  }
}
```

## Interview Questions
1. **What is the difference between Promise and Async/Await?**
   Async/Await is cleaner and easier to read, especially when dealing with multiple sequential async operations. It also allows using try/catch for error handling.
2. **What happens if you don't use 'await' inside an 'async' function?**
   The function will still return a promise, but it will execute synchronously until it reaches an async operation or the end of the function.
