- Category: Core Concepts
- Difficulty: Beginner
- Related: closures

### What is Hoisting?
Hoisting is JavaScript's default behavior of moving declarations to the top of the current scope before code execution.

---

### 1. Execution Context
**Theory**: During the compilation phase, JS stores function and variable declarations in memory, which allows you to use them before you declare them in the code.

**Working Flow**
```text
[ Compile Phase ] --> ( Move declarations to top ) --> [ Execution Phase ]
```

**Key Features**:
- **var**: Hoisted and initialized with `undefined`.
- **let/const**: Hoisted but NOT initialized (Temporal Dead Zone).
- **Functions**: Fully hoisted (can be called before definition).

**Example**:
```javascript
// Function Hoisting
greet(); // Works perfectly!
function greet() { console.log("Hello!"); }

// Variable Hoisting
console.log(myVar); // undefined (not an error)
var myVar = 10;
```

---

[View Interview Questions](./interview.md)
