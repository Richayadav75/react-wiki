- Category: Core Concepts
- Difficulty: Intermediate
- Related: closures

### What is the "this" Keyword?
The `this` keyword refers to the object that is currently executing the code. Its value depends entirely on *how* the function is called.

---

### 1. Execution Context Binding
**Theory**: Unlike other languages, `this` is not bound to the function itself, but to the object calling it.

**Key Features**:
- **Global Context**: Refers to the `window` object.
- **Object Method**: Refers to the object owning the method.
- **Arrow Functions**: They DO NOT have their own `this`; they inherit it from the parent scope.

**Example**:
```javascript
const user = {
  name: "Richa",
  greet: function() {
    console.log("Hi, I am " + this.name); // "this" is the user object
  }
};
user.greet();
```

---

[View Interview Questions](./interview.md)
