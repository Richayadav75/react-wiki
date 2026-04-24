- Category: JavaScript
- Difficulty: Intermediate
- Related: hoisting

### What are Closures?
A closure is a function that remembers its outer variables and can access them even after the outer function has finished executing.

---

### 1. Lexical Scoping
**Theory**: Functions in JavaScript are bound to their lexical environment. They "remember" the scope in which they were created.

**Working Flow**
```text
[ Outer Function Scope ] <--- ( Accesses outer data ) --- [ Inner Function ]
```

**Key Features**:
- **Data Privacy**: Encapsulates variables so they cannot be accessed from the global scope.
- **State Retention**: Remembers the state across multiple invocations.

**Example**:
```javascript
function makeCounter() {
  let count = 0; // Private variable
  return function() {
    count++; // Accesses outer scope
    return count;
  };
}

let counter = makeCounter();
console.log(counter()); // 1
console.log(counter()); // 2
```
