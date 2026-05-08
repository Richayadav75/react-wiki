# ES6+ Features — Interview Questions

---

**1. What is the difference between `let`, `const`, and `var`?**

| Feature | `var` | `let` | `const` |
|---------|-------|-------|---------|
| Scope | Function | Block | Block |
| Hoisted | Yes (as `undefined`) | Yes (TDZ — unusable) | Yes (TDZ — unusable) |
| Re-declare | Yes | No | No |
| Re-assign | Yes | Yes | No |

```javascript
var a = 1;  var a = 2;    // OK
let b = 1;  // let b = 2; // SyntaxError
const c = 1; // c = 2;   // TypeError
```

Use `const` by default, `let` when you need to reassign, avoid `var`.

---

**2. What is Destructuring? Give examples for both objects and arrays.**

Destructuring extracts values from objects or arrays into variables in one statement.

```javascript
// Object
const { name, age = 18 } = { name: "Alice" };
console.log(name, age); // → "Alice"  18

// Rename
const { name: fullName } = { name: "Bob" };
console.log(fullName); // → "Bob"

// Array
const [first, , third] = [10, 20, 30];
console.log(first, third); // → 10  30

// Swap
let x = 1, y = 2;
[x, y] = [y, x];
console.log(x, y); // → 2  1
```

---

**3. What is the difference between Spread and Rest?**

Both use `...` but work in opposite directions:

```javascript
// Spread — EXPANDS an iterable into elements
const arr = [1, 2, 3];
const copy = [...arr, 4, 5];     // [1,2,3,4,5]
const obj  = { ...{a:1}, b:2 };  // {a:1,b:2}

// Rest — COLLECTS remaining elements into an array
function sum(a, b, ...rest) {
  console.log(rest); // [3,4,5]
  return a + b + rest.reduce((t, n) => t + n, 0);
}
sum(1, 2, 3, 4, 5); // → 15
```

---

**4. What are Template Literals and when should you use them?**

Template literals use backticks and support expression interpolation (`${}`), multi-line strings, and method calls.

```javascript
const name = "Alice";
const score = 92;

console.log(`${name} scored ${score}% — ${score >= 60 ? "Pass" : "Fail"}`);
// → "Alice scored 92% — Pass"

const html = `
  <div class="card">
    <h2>${name}</h2>
    <p>Score: ${score}</p>
  </div>
`;
```

Use them whenever you are building strings with variables. Avoid `+` string concatenation.

---

**5. What are Arrow Functions? When should you NOT use them?**

Arrow functions are concise and have lexical `this`. They also have implicit return for single expressions.

```javascript
const add = (a, b) => a + b;
const square = n => n * n;
const makeObj = id => ({ id, ts: Date.now() }); // wrap object in ()
```

Do NOT use arrow functions as:
- Object methods (they won't bind to the object's `this`)
- Constructor functions (cannot use `new` with them)
- Functions that need the `arguments` object

---

**6. What is Optional Chaining (`?.`) and why is it important?**

`?.` short-circuits and returns `undefined` instead of throwing a `TypeError` when accessing a property on `null` or `undefined`.

```javascript
const user = { profile: { city: "Mumbai" } };
const noProfile = { name: "Bob" };

console.log(user?.profile?.city);      // → "Mumbai"
console.log(noProfile?.profile?.city); // → undefined  (no crash)

// Without it:
// noProfile.profile.city  → TypeError: Cannot read properties of undefined
```

Essential when working with API responses where nested data might be absent.

---

**7. What is the Nullish Coalescing operator (`??`) and how is it different from `||`?**

`??` returns the right side only if the left side is `null` or `undefined`. `||` returns the right side for any falsy value (`0`, `""`, `false`, `null`, `undefined`).

```javascript
const count = 0;
const name  = "";

console.log(count || 10);  // → 10  (WRONG — 0 is falsy!)
console.log(count ?? 10);  // → 0   (CORRECT — 0 is not null/undefined)

console.log(name  || "Anonymous");  // → "Anonymous" (may be wrong)
console.log(name  ?? "Anonymous");  // → ""          (keeps empty string)
```

Use `??` for default values when `0`, `""`, or `false` are legitimate values.

---

**8. What is the difference between named exports and default exports?**

```javascript
// Named — can export multiple, import with exact name or alias
export const PI = 3.14;
export function add(a, b) { return a + b; }

import { PI, add } from "./math.js";
import { add as sum } from "./math.js";

// Default — one per file, import with any name
export default class Calculator { ... }

import Calculator from "./math.js";     // any name works
import Calc from "./math.js";           // same default
```

Named exports are better for utilities. Default exports are common for React components.

---

**9. What is shorthand property notation? Give a real example.**

When a variable's name matches the object key you want to create, you can write it once:

```javascript
const name = "Alice";
const age  = 28;
const role = "admin";

// Old
const user = { name: name, age: age, role: role };

// Shorthand (ES6)
const user = { name, age, role };

// Common in functions returning data
function createProduct(id, price, stock) {
  return { id, price, stock, available: stock > 0 };
}
createProduct(1, 999, 5);
// → { id:1, price:999, stock:5, available:true }
```

---

**10. How do you merge two objects and handle key conflicts in ES6?**

Use spread. Later properties override earlier ones:

```javascript
const defaults = { theme: "light", lang: "en", fontSize: 16 };
const userPrefs = { theme: "dark", fontSize: 20 };

const settings = { ...defaults, ...userPrefs };
// → { theme:"dark", lang:"en", fontSize:20 }
// "theme" and "fontSize" from userPrefs win

// Deep merge requires recursion or a library (lodash.merge)
// Spread is SHALLOW — nested objects are NOT deep-merged
const a = { x: { y: 1 } };
const b = { x: { z: 2 } };
const merged = { ...a, ...b };
// → { x: { z: 2 } }  ← a.x.y is LOST (b.x replaced a.x entirely)
```
