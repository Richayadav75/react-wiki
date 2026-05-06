- Category: Fundamentals
- Track: Fundamentals
- Difficulty: Beginner
- Related: data-types

### What are Variables?
Variables are named storage for data. In modern JavaScript, we have three distinct ways to create them, each with unique behaviors.

---

### 1. var (The Legacy Way)
**Theory**: Before 2015, var was the only way to declare a variable. It has function scope and gets hoisted — both cause tricky bugs. Avoid in modern code.


**Key Features**:
- **Scope**: Function-scoped (accessible anywhere in the function).
- **Hoisting**: Moved to the top of its scope during execution.
- **Re-declare**: You can declare the same name twice without error.

**Example**:
```javascript
var name = "Alice";   // declaring a variable
var age  = 25;        // storing a number
var active = true;    // storing a boolean

console.log(name);    // → Alice
console.log(age);     // → 25
console.log(active);  // → true

```

**Explanation**:

`var name = "Alice";`
Keyword var, then the name, then = to assign, then the value.

`var age = 25;`
No quotes → number. With quotes → string.

`var active = true;`
true and false (no quotes) are boolean values.

`console.log(name);`
Prints the value to the browser console.

**Output**:
```
Alice
25
true
```
---

### 2. let (The Modern Standard)
**Theory**: Introduced in ES6 to fix the issues with `var`. It is block-scoped, meaning it only exists within `{ }`.

let is the modern replacement for var when a value might change. It is scoped to the nearest {} block, so it's safer and more predictable.



**Key Features**:
- **Scope**: Block-scoped (limited to the nearest curly braces).
- **Updateable**: You can change the value, but NOT re-declare the name.
- **No Hoisting**: You cannot use it before declaration.

**Example**:
```javascript
let score = 0;          // starts at 0
score = 10;             // we can update it
let message = "hello";
message = "world";      // also updatable

console.log(score);     // → 10
console.log(message);   // → world
```

**Output**:
```
10
world
```

**Explanation**:

`let score = 0;`
Declare with let. Value can change later.

`score = 10;`
Reassignment — no keyword needed, just variable = newValue.

`message = "world";`
Strings can be reassigned the same way.
---

### 3. const (The Constant)
**Theory**: Used for values that should never change throughout the program.
const means the binding cannot be reassigned. Use it by default — switch to let only when you know the value will change.

**Key Features**:
- **Fixed Value**: Once assigned, it cannot be changed.
- **Block-scoped**: Like `let`, it lives only within `{ }`.
- **Initialization**: You MUST assign a value immediately.

**Example**:
```javascript
const PI = 3.14159;       // mathematical constant
const userName = "Bob";   // won't change

// PI = 3; ← this would throw a TypeError!

const colors = ["red", "blue"];
colors.push("green");     // arrays CAN be mutated
console.log(colors);      // → ["red","blue","green"]
```

**Explanation**:

`const PI = 3.14159;`
Constant — trying to reassign throws a TypeError at runtime.

`const colors = ["red", "blue"];`
The array reference is constant, but its contents can change.

`colors.push("green");`
.push adds to the array — this is mutation, not reassignment.

---

[View Interview Questions](./interview.md)
