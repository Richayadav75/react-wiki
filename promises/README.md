- Category: JavaScript
- Difficulty: Intermediate
- Related: Async/Await, Callbacks

# Promises

The `Promise` object represents the eventual completion (or failure) of an asynchronous operation and its resulting value.

## States
1. **Pending**: Initial state, neither fulfilled nor rejected.
2. **Fulfilled**: The operation completed successfully.
3. **Rejected**: The operation failed.

## Example
```javascript
const myPromise = new Promise((resolve, reject) => {
  const success = true;
  if (success) {
    resolve("Operation successful!");
  } else {
    reject("Operation failed.");
  }
});

myPromise
  .then(value => console.log(value))
  .catch(error => console.error(error));
```

## Chaining
Promises allow you to chain operations using `.then()`, avoiding "callback hell".
