- Category: Fundamentals
- Track: Fundamentals
- Difficulty: Beginner
- Related: variables

### What are Data Types?
Data types define the nature of data being stored. JavaScript has **Primitives** and **Reference** types.

---

### 1. Primitive Types
**Theory**: Primitives are the most basic data types in JavaScript. They are immutable, meaning once assigned, they cannot be changed. If you try to modify a primitive variable, you are actually creating a new variable with the new value.  
The most basic, unchangeable data types stored by value.



**Key Features**:
- **Number**: Integers and decimals (`10`, `3.14`).
- **String**: Text data wrapped in quotes (`"Hello"`).
- **Boolean**: Logical values (`true`, `false`).
- **Null**: Intentional empty value.
- **Undefined**: Value has not been assigned.
- **Symbol**: Unique identifier.

**Example**:
```javascript
let str  = "hello";          // String
let num  = 42;               // Number
let dec  = 3.14;             // Number (decimals too)
let bool = true;             // Boolean
let nothing = null;          // Null (intentional empty)
let missing = undefined;     // Undefined (not assigned)
let id = Symbol("uid");      // Symbol (unique identifier)

console.log(typeof str);     // → "string"
console.log(typeof num);     // → "number"
console.log(typeof bool);    // → "boolean"
console.log(typeof nothing); // → "object" ← JS quirk!
console.log(typeof missing); // → "undefined"

let str = "hello";
String — text wrapped in quotes (single or double).
let num = 42; let dec = 3.14;
Number — integers and decimals share the same type in JS.
let bool = true;
Boolean — only two values: true or false.
let nothing = null;
Null — you set this intentionally to mean 'no value'.
let missing = undefined;
Undefined — variable declared but never assigned.
typeof nothing // → 'object'
Famous JS bug — typeof null returns 'object'. It's a historical mistake.
```
**Output**:
```
string
number
boolean
object
undefined
```
---

### 2. Reference Types
**Theory**: Complex structures that can store multiple values and are stored by reference in memory.



**Key Features**:
- **Objects**: Collections of key-value pairs.
- **Arrays**: Ordered lists of data.

**Example**:
```javascript
let user = { name: "Richa", city: "Delhi" };
let colors = ["Red", "Blue", "Green"];
```

---

[View Interview Questions](./interview.md)
