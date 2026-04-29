- Category: Fundamentals
- Track: Fundamentals
- Difficulty: Beginner
- Related: data-types

### What are Variables?
Variables are named storage for data. In modern JavaScript, we have three distinct ways to create them, each with unique behaviors.

---

### 1. var (The Legacy Way)
**Theory**: Before ES6, `var` was the only way to declare variables. It is function-scoped and allows re-declaration.

**Key Features**:
- **Scope**: Function-scoped (accessible anywhere in the function).
- **Hoisting**: Moved to the top of its scope during execution.
- **Re-declare**: You can declare the same name twice without error.

**Example**:
```javascript
var name = "Richa";
var name = "Yadav"; // No error, it just overwrites
console.log(name); // Yadav
```

---

### 2. let (The Modern Standard)
**Theory**: Introduced in ES6 to fix the issues with `var`. It is block-scoped, meaning it only exists within `{ }`.

**Key Features**:
- **Scope**: Block-scoped (limited to the nearest curly braces).
- **Updateable**: You can change the value, but NOT re-declare the name.
- **No Hoisting**: You cannot use it before declaration.

**Example**:
```javascript
let score = 10;
score = 20; // Allowed: Update value
// let score = 30; // ERROR! Cannot re-declare
console.log(score); // 20
```

---

### 3. const (The Constant)
**Theory**: Used for values that should never change throughout the program.

**Key Features**:
- **Fixed Value**: Once assigned, it cannot be changed.
- **Block-scoped**: Like `let`, it lives only within `{ }`.
- **Initialization**: You MUST assign a value immediately.

**Example**:
```javascript
const PI = 3.14;
// PI = 4; // ERROR! Constant cannot change
console.log(PI);
```

---

[View Interview Questions](./interview.md)
