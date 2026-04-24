- Category: JavaScript
- Difficulty: Intermediate
- Related: async-await

### What are Promises?
A Promise is an object representing the eventual completion (or failure) of an asynchronous operation.

---

### 1. Asynchronous Handling
**Theory**: Instead of using callbacks (which lead to callback hell), promises provide a cleaner, chainable way to handle async tasks.

**Working Flow**
```text
[ Pending ] --> ( Success ) --> [ Resolved (.then) ]
            |
            L-> ( Failure ) --> [ Rejected (.catch) ]
```

**Key Features**:
- **States**: Pending, Fulfilled, or Rejected.
- **Chaining**: You can chain multiple `.then()` calls.
- **Error Catching**: A single `.catch()` handles errors for the chain.

**Example**:
```javascript
let myPromise = new Promise((resolve, reject) => {
  let success = true;
  if(success) resolve("Task Complete!");
  else reject("Task Failed!");
});

myPromise
  .then((result) => console.log(result))
  .catch((error) => console.log(error));
```
