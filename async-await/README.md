- Category: JavaScript
- Difficulty: Intermediate
- Related: promises

### What is Async/Await?
Async/Await is syntactic sugar over Promises, making asynchronous code look and behave more like synchronous code.

---

### 1. Modern Async Syntax
**Theory**: By marking a function as `async`, you can use the `await` keyword inside it to pause execution until a Promise resolves.

**Working Flow**
```text
[ Call Async Func ] --> ( Hit Await ) --> [ Pause Func, Run Other Code ] --> ( Promise Resolves ) --> [ Resume Func ]
```

**Key Features**:
- **Readable**: Looks like top-down synchronous code.
- **Error Handling**: Uses standard `try...catch` blocks instead of `.catch()`.

**Example**:
```javascript
async function fetchData() {
  try {
    let response = await fetch('https://api.example.com/data');
    let data = await response.json();
    console.log(data);
  } catch (error) {
    console.log("Error:", error);
  }
}
```
