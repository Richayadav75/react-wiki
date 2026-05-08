- Category: JavaScript
- Difficulty: Beginner
- Related: variables, array-methods, this-keyword, functions

### ES6+ Features — Modern JavaScript
ES6 (ECMAScript 2015) was the biggest upgrade to JavaScript since its creation. Every feature listed here is used daily in real React/Node projects. Think of ES6 as JavaScript growing up — replacing verbose, error-prone patterns with concise, readable ones.

**Analogy**
Pre-ES6 JavaScript was like sending text messages on a T9 keypad — functional but painful. ES6 is the smartphone keyboard: faster, clearer, and you rarely make mistakes.

---

### 1. let / const vs var — Block Scoping
**Theory**: `var` is function-scoped and hoisted (moved to the top with value `undefined`). `let` and `const` are **block-scoped** (live only inside `{}`), not hoisted in the same accessible way (they sit in a Temporal Dead Zone until declared).

**Working Flow**
![flow-chart](flow-chart.png)

**Example**
```javascript
// var — hoisted, function-scoped (dangerous)
console.log(a); // → undefined (hoisted, not error)
var a = 5;

for (var i = 0; i < 3; i++) {}
console.log(i); // → 3  (leaked out of loop!)

// let — block-scoped, not accessible before declaration
// console.log(b); // → ReferenceError (TDZ)
let b = 10;

for (let j = 0; j < 3; j++) {}
// console.log(j); // → ReferenceError (j is gone)

// const — block-scoped, cannot be reassigned
const PI = 3.14159;
// PI = 3;  // → TypeError: Assignment to constant variable

// const with objects — object itself is mutable
const user = { name: "Alice" };
user.name = "Bob";   // ✓ — property can change
user.age  = 28;      // ✓ — can add properties
// user = {};        // ✗ — cannot reassign the binding
console.log(user);   // → { name: "Bob", age: 28 }
```

**Output**
```
console.log(a)  → undefined  (var hoisted)
i after loop    → 3          (var leaked)
user after edit → { name: "Bob", age: 28 }
```

**Explanation**: Use `const` by default. Use `let` when you need to reassign. Avoid `var` in modern code.

---

### 2. Template Literals — Readable Strings
**Theory**: Template literals use backticks (\`) instead of quotes. They support embedded expressions (`${}`), multi-line strings without `\n`, and tagged templates for advanced formatting.

**Working Flow**
![flow-chart-2](flow-chart-2.png)

**Example**
```javascript
const name = "Alice";
const age  = 28;
const score = 87.5;

// Basic interpolation
console.log(`Hello ${name}, you are ${age} years old.`);
// → "Hello Alice, you are 28 years old."

// Expressions inside ${}
console.log(`Score: ${score}% — ${score >= 60 ? "Pass" : "Fail"}`);
// → "Score: 87.5% — Pass"

// Multi-line (no \n needed)
const card = `
  Name:  ${name}
  Age:   ${age}
  Score: ${score}
`;

// Function calls inside
const items = ["apple", "banana", "mango"];
console.log(`You have ${items.length} items: ${items.join(", ")}`);
// → "You have 3 items: apple, banana, mango"
```

**Output**
```
`Hello ${name}...`       → "Hello Alice, you are 28 years old."
`Score: ${score}% — ...` → "Score: 87.5% — Pass"
`You have ${items.length}...` → "You have 3 items: apple, banana, mango"
```

---

### 3. Destructuring — Unpack Values Cleanly
**Theory**: Destructuring lets you extract values from objects or arrays into named variables in a single statement. Supports defaults, renaming, and nested unpacking.

**Working Flow**
![flow-chart-3](flow-chart-3.png)

**Example**
```javascript
// --- Object Destructuring ---
const config = { host: "localhost", port: 3000, debug: true };

const { host, port }             = config;           // basic
const { debug: isDebug }         = config;           // rename
const { timeout = 5000 }         = config;           // default (not in object)
const { host: h, port: p = 80 }  = config;           // rename + default

console.log(host);    // → "localhost"
console.log(isDebug); // → true
console.log(timeout); // → 5000  (default used)

// Destructure function parameter
function connect({ host, port = 80, secure = false }) {
  console.log(`${secure ? "https" : "http"}://${host}:${port}`);
}
connect({ host: "api.example.com", secure: true });
// → "https://api.example.com:80"

// --- Array Destructuring ---
const coords = [40.7128, -74.0060, 10];

const [lat, lng]         = coords;    // basic
const [,, altitude]      = coords;    // skip elements with commas
const [first, ...rest]   = [1,2,3,4]; // rest element

console.log(lat, lng);   // → 40.7128 -74.006
console.log(altitude);   // → 10
console.log(rest);       // → [2, 3, 4]

// Swap two variables
let a = 1, b = 2;
[a, b] = [b, a];
console.log(a, b); // → 2 1
```

**Output**
```
host             → "localhost"
isDebug          → true
timeout          → 5000
connect(...)     → "https://api.example.com:80"
lat, lng         → 40.7128  -74.006
altitude         → 10
rest             → [2,3,4]
a, b after swap  → 2  1
```

---

### 4. Spread & Rest Operators (`...`)
**Theory**: The `...` operator has two roles depending on context.
- **Spread**: *expands* an iterable (array, object, string) into individual elements.
- **Rest**: *collects* remaining elements into a single array or object.

**Working Flow**
![flow-chart-4](flow-chart-4.png)

**Example**
```javascript
// --- Spread ---
const a = [1, 2, 3];
const b = [4, 5, 6];

const combined  = [...a, ...b];          // → [1,2,3,4,5,6]
const withExtra = [0, ...a, 3.5, ...b];  // → [0,1,2,3,3.5,4,5,6]
const copy      = [...a];                // shallow clone

// Spread with objects
const defaults = { theme: "light", lang: "en", fontSize: 16 };
const overrides = { theme: "dark", fontSize: 18 };
const settings  = { ...defaults, ...overrides }; // later keys win
// → { theme:"dark", lang:"en", fontSize:18 }

// Spread into function args
const nums = [3, 1, 4, 1, 5, 9];
console.log(Math.max(...nums)); // → 9

// --- Rest ---
function sum(first, second, ...others) {
  console.log(first);   // → 1
  console.log(second);  // → 2
  console.log(others);  // → [3, 4, 5]
  return first + second + others.reduce((t, n) => t + n, 0);
}
console.log(sum(1, 2, 3, 4, 5)); // → 15

// Rest in destructuring
const [head, ...tail] = [10, 20, 30, 40];
console.log(head); // → 10
console.log(tail); // → [20, 30, 40]

// Rest in object destructuring
const { id, ...profile } = { id: 1, name: "Alice", role: "admin" };
console.log(id);      // → 1
console.log(profile); // → { name:"Alice", role:"admin" }
```

**Output**
```
combined          → [1,2,3,4,5,6]
settings          → {theme:"dark", lang:"en", fontSize:18}
Math.max(...nums) → 9
sum(1,2,3,4,5)    → 15
head              → 10
tail              → [20,30,40]
profile           → {name:"Alice",role:"admin"}
```

---

### 5. Arrow Functions
**Theory**: Arrow functions are a shorter way to write functions. Key differences from regular functions: no `this` binding (inherits lexically), no `arguments` object, cannot be used as constructors, implicit return for single expressions.

**Working Flow**
![flow-chart-5](flow-chart-5.png)

**Example**
```javascript
// Equivalent forms
const double = function(n) { return n * 2; };
const double2 = n => n * 2;

// No params
const getRandom = () => Math.random();

// Multi-line — needs return
const clamp = (val, min, max) => {
  if (val < min) return min;
  if (val > max) return max;
  return val;
};

// Returning object literal — wrap in parentheses
const makeUser = (name, age) => ({ name, age, active: true });
console.log(makeUser("Alice", 28));
// → { name:"Alice", age:28, active:true }

// Arrow functions in array methods (very common)
const nums  = [1, 2, 3, 4, 5];
const even  = nums.filter(n => n % 2 === 0);   // [2, 4]
const sq    = nums.map(n => n * n);             // [1, 4, 9, 16, 25]
const total = nums.reduce((s, n) => s + n, 0); // 15
```

**Output**
```
double2(5)      → 10
makeUser(...)   → {name:"Alice",age:28,active:true}
even            → [2,4]
sq              → [1,4,9,16,25]
total           → 15
```

---

### 6. Default Parameters & Shorthand Properties
**Theory**: Default parameters let you specify fallback values for function arguments. Shorthand properties let you write `{ name }` instead of `{ name: name }` when the variable name matches the key.

**Example**
```javascript
// Default parameters
function greet(name = "Guest", greeting = "Hello") {
  return `${greeting}, ${name}!`;
}
console.log(greet());              // → "Hello, Guest!"
console.log(greet("Alice"));      // → "Hello, Alice!"
console.log(greet("Bob", "Hey")); // → "Hey, Bob!"

// Works with destructuring defaults too
function connect({ host = "localhost", port = 3000 } = {}) {
  return `${host}:${port}`;
}
connect();                       // → "localhost:3000"
connect({ port: 8080 });         // → "localhost:8080"

// Shorthand properties
const name = "Alice";
const age  = 28;

// Old way
const userOld = { name: name, age: age };

// Shorthand (ES6)
const user = { name, age };
console.log(user); // → { name:"Alice", age:28 }

// Shorthand methods
const calc = {
  value: 0,
  add(n)      { this.value += n; return this; },
  multiply(n) { this.value *= n; return this; },
  result()    { return this.value; }
};
console.log(calc.add(5).multiply(3).result()); // → 15
```

**Output**
```
greet()              → "Hello, Guest!"
greet("Alice")       → "Hello, Alice!"
greet("Bob","Hey")   → "Hey, Bob!"
connect()            → "localhost:3000"
user                 → {name:"Alice",age:28}
calc chain           → 15
```

---

### 7. Optional Chaining (?.) and Nullish Coalescing (??)
**Theory**: Two ES2020 features that make working with potentially missing data safe and clean.
- `?.` short-circuits and returns `undefined` instead of throwing if a property doesn't exist.
- `??` returns the right-hand value only when the left-hand is `null` or `undefined` (not for `0`, `""`, `false`).

**Working Flow**
![flow-chart-6](flow-chart-6.png)

**Example**
```javascript
const user = {
  name: "Alice",
  address: {
    city: "Mumbai",
    zip: "400001"
  },
  getScore: () => 95
};

const noAddress = { name: "Bob" };

// Optional chaining
console.log(user?.address?.city);      // → "Mumbai"
console.log(noAddress?.address?.city); // → undefined  (no crash!)
console.log(user?.phone?.number);      // → undefined

// Method calls
console.log(user?.getScore?.());       // → 95
console.log(noAddress?.getScore?.());  // → undefined

// Array items
const arr = null;
console.log(arr?.[0]); // → undefined

// Nullish coalescing
const config = { timeout: 0, name: "", retries: null };

console.log(config.timeout  ?? 5000); // → 0       (0 is NOT null/undefined)
console.log(config.name     ?? "app");// → ""      ("" is NOT null/undefined)
console.log(config.retries  ?? 3);    // → 3       (null → use default)
console.log(config.missing  ?? "N/A");// → "N/A"   (undefined → use default)

// Compare with || (dangerous with falsy values)
console.log(config.timeout  || 5000); // → 5000    (WRONG! 0 is falsy)
console.log(config.timeout  ?? 5000); // → 0       (CORRECT)
```

**Output**
```
user?.address?.city       → "Mumbai"
noAddress?.address?.city  → undefined
user?.getScore?.()        → 95
config.timeout ?? 5000    → 0
config.retries ?? 3       → 3
config.timeout || 5000    → 5000  ← (wrong!)
config.timeout ?? 5000    → 0     ← (correct)
```

---

### 8. Modules — import / export
**Theory**: ES6 modules replace the old global-script approach. Each file is its own scope. You explicitly `export` what should be public and `import` what you need. Named exports allow multiple per file; default export allows one per file.

**Working Flow**
![flow-chart-7](flow-chart-7.png)

**Example**
```javascript
// --- utils.js ---
// Named exports
export const VERSION = "1.0.0";

export function formatDate(date) {
  return date.toLocaleDateString("en-IN");
}

export const slugify = (str) =>
  str.toLowerCase().trim().replace(/\s+/g, "-");

// Default export
export default class Logger {
  static log(msg) { console.log(`[LOG] ${msg}`); }
}

// --- app.js ---
import Logger, { VERSION, formatDate, slugify } from "./utils.js";

Logger.log(VERSION);                          // → [LOG] 1.0.0
console.log(slugify("Hello World"));          // → "hello-world"
console.log(formatDate(new Date("2024-03-15")));// → "15/3/2024"

// Rename on import
import { formatDate as fmtDate } from "./utils.js";

// Import everything as namespace
import * as Utils from "./utils.js";
Utils.slugify("Test String"); // → "test-string"
```

**Output**
```
Logger.log(VERSION)    → [LOG] 1.0.0
slugify("Hello World") → "hello-world"
fmtDate(...)           → "15/3/2024"
Utils.slugify(...)     → "test-string"
```

---

### Real-World Practical Example — API Response Transformer
```javascript
// Using destructuring, spread, optional chaining, nullish coalescing
function transformUser(apiUser) {
  const {
    id,
    full_name: name,
    email,
    profile: {
      avatar_url: avatar = "/default-avatar.png",
      bio = "No bio provided"
    } = {},
    role = "viewer"
  } = apiUser;

  return {
    id,
    name,
    email,
    avatar,
    bio,
    role,
    displayName: name ?? email ?? `User #${id}`,
    isAdmin: role === "admin"
  };
}

const rawUser = {
  id: 42,
  full_name: "Alice Chen",
  email: "alice@example.com",
  profile: { avatar_url: "https://cdn.example.com/alice.jpg" },
  role: "admin"
};

console.log(transformUser(rawUser));
```

**Output**
```
{
  id: 42,
  name: "Alice Chen",
  email: "alice@example.com",
  avatar: "https://cdn.example.com/alice.jpg",
  bio: "No bio provided",
  role: "admin",
  displayName: "Alice Chen",
  isAdmin: true
}
```

---

[View Interview Questions](./interview.md)
