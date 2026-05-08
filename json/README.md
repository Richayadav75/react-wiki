- Category: JavaScript
- Difficulty: Beginner
- Related: object-methods, restful-api, async-await

### JSON — The Universal Data Language
**JSON** (JavaScript Object Notation) is a lightweight, text-based format for representing structured data. It is not JavaScript code — it is a plain string that happens to look like a JavaScript object. Every major programming language can read and write it, making it the standard format for API communication and data storage.

**Analogy**
Think of JSON as a shipping container. Your JavaScript object is the cargo — it lives in memory, can have functions, and behaves. JSON is the flat-packed box you use to ship that cargo over the internet. You pack it (`JSON.stringify`), ship it (HTTP), and unpack it at the other end (`JSON.parse`). Functions and complex types can't fit in the box — data only.

---

### 1. JSON vs JavaScript Object

**Theory**: A JavaScript object lives in memory as a data structure. A JSON string is just text — it has no methods, no `undefined`, no functions. The two are related in shape but are different things.

**Working Flow**
![flow-chart](flow-chart.png)

**Example**
```javascript
// JS Object — keys can be unquoted, can have functions
const jsObj = {
  name: "Alice",
  age: 30,
  active: true,
  score: null,
  greet() { return "hello"; },   // function — will be dropped
  status: undefined,              // undefined — will be dropped
};

// JSON string — double-quoted keys, no functions
const jsonStr = '{"name":"Alice","age":30,"active":true,"score":null}';

// They are NOT the same type
console.log(typeof jsObj);    // "object"
console.log(typeof jsonStr);  // "string"

// JSON Rules:
// ✓ strings (double quotes only), numbers, boolean, null, objects, arrays
// ✗ undefined, functions, Symbol, Date (becomes ISO string), BigInt (throws)
```

**Output**
```
typeof jsObj    → "object"
typeof jsonStr  → "string"
greet()         → dropped by stringify
undefined       → dropped by stringify
```

---

### 2. JSON.stringify — Object to String

**Theory**: `JSON.stringify(value, replacer, space)` converts a JavaScript value to a JSON string.
- `replacer` — an array of keys to include, or a function to transform values
- `space` — number of spaces (or a string) to use for pretty-printing

**Working Flow**
![flow-chart-2](flow-chart-2.png)

**Example**
```javascript
const user = {
  name:   "Alice",
  age:    30,
  scores: [95, 88, 72],
  address: { city: "Mumbai", zip: "400001" },
  password: "secret",     // sensitive — we want to exclude
  greet() {},             // function — auto dropped
};

// Basic
console.log(JSON.stringify(user));
// '{"name":"Alice","age":30,"scores":[95,88,72],"address":{...},"password":"secret"}'

// Pretty print (space = 2)
console.log(JSON.stringify(user, null, 2));
// {
//   "name": "Alice",
//   "age": 30,
//   ...
// }

// Replacer as array — only include specific keys
console.log(JSON.stringify(user, ["name", "age"]));
// '{"name":"Alice","age":30}'

// Replacer as function — exclude sensitive fields
const safe = JSON.stringify(user, (key, value) => {
  if (key === "password") return undefined; // exclude
  return value;
});
console.log(safe);
// '{"name":"Alice","age":30,"scores":[...],"address":{...}}'
```

**Output**
```
basic stringify      → {...,"password":"secret","greet" dropped}
pretty (space:2)     → multi-line indented JSON
replacer ["name","age"] → '{"name":"Alice","age":30}'
replacer fn (no pwd)    → password excluded
```

---

### 3. JSON.parse — String to Object

**Theory**: `JSON.parse(string, reviver)` converts a JSON string back into a JavaScript value.
- `reviver` — a function that transforms each key-value pair as it is parsed (useful for converting date strings back to Date objects)

**Working Flow**
![flow-chart-3](flow-chart-3.png)

**Example**
```javascript
const jsonStr = '{"name":"Alice","age":30,"joinDate":"2023-01-15","scores":[95,88]}';

// Basic parse
const obj = JSON.parse(jsonStr);
console.log(obj.name);         // "Alice"
console.log(obj.scores[0]);    // 95
console.log(typeof obj.age);   // "number"

// Reviver — transform values during parsing
const withDate = JSON.parse(jsonStr, (key, value) => {
  if (key === "joinDate") return new Date(value); // string → Date
  return value;
});
console.log(withDate.joinDate instanceof Date); // true
console.log(withDate.joinDate.getFullYear());   // 2023

// Error handling — invalid JSON throws SyntaxError
try {
  JSON.parse("{ invalid json }");
} catch (e) {
  console.log("Parse error:", e.message); // "Unexpected token i..."
}
```

**Output**
```
obj.name               → "Alice"
obj.scores[0]          → 95
typeof obj.age         → "number"
withDate.joinDate      → Date object (instanceof Date = true)
joinDate.getFullYear() → 2023
parse invalid JSON     → SyntaxError
```

---

### 4. What Gets Dropped by stringify

**Theory**: Several JavaScript types cannot be represented in JSON. `JSON.stringify` silently drops them when they appear as object property values. When they appear as a top-level value or array element, they become `null`.

**Working Flow**
![flow-chart-4](flow-chart-4.png)

**Example**
```javascript
const data = {
  name:     "test",
  nothing:  undefined,        // dropped
  fn:       function() {},    // dropped
  sym:      Symbol("s"),      // dropped
  nan:      NaN,              // → null
  inf:      Infinity,         // → null
  now:      new Date(),       // → ISO string
};

const result = JSON.stringify(data);
console.log(result);
// {"name":"test","nan":null,"inf":null,"now":"2024-01-15T..."}

// In arrays — undefined/fn/Symbol → null
const arr = [1, undefined, function(){}, null, Symbol("x")];
console.log(JSON.stringify(arr));
// [1,null,null,null,null]

// BigInt throws
try {
  JSON.stringify({ big: 9007199254740993n });
} catch(e) {
  console.log(e.message); // "Do not know how to serialize a BigInt"
}
```

**Output**
```
object stringify:
  nothing (undefined) → key dropped
  fn (function)       → key dropped
  sym (Symbol)        → key dropped
  nan (NaN)           → null
  inf (Infinity)      → null
  now (Date)          → "2024-01-15T..."

array stringify:
  [1, undefined, fn, null, Symbol] → [1,null,null,null,null]

BigInt → TypeError
```

---

### 5. Deep Clone via JSON

**Theory**: `JSON.parse(JSON.stringify(obj))` creates a completely independent copy of a nested object — breaking all reference links. This is quick and readable, but has limitations: it drops functions, converts Dates to strings, and fails on circular references.

**Working Flow**
![flow-chart-5](flow-chart-5.png)

**Example**
```javascript
const original = {
  user: "Alice",
  prefs: { theme: "dark", fontSize: 14 },
  tags: ["js", "react"],
};

// Shallow copy — nested objects still shared
const shallow = { ...original };
shallow.prefs.theme = "light";
console.log(original.prefs.theme); // "light" — AFFECTED!

// Deep clone via JSON
const original2 = {
  user: "Bob",
  prefs: { theme: "dark", fontSize: 14 },
  tags: ["js", "react"],
};
const deep = JSON.parse(JSON.stringify(original2));
deep.prefs.theme = "light";
deep.tags.push("vue");
console.log(original2.prefs.theme); // "dark" — SAFE!
console.log(original2.tags);        // ["js","react"] — SAFE!

// Limitations
const withFn = { name: "test", fn: () => 42, date: new Date() };
const cloned  = JSON.parse(JSON.stringify(withFn));
console.log(cloned.fn);             // undefined (function dropped)
console.log(typeof cloned.date);    // "string" (Date → string)
```

**Output**
```
shallow copy:
  original.prefs.theme    → "light" (mutated — shared reference)

deep clone:
  original2.prefs.theme   → "dark"  (safe — new reference)
  original2.tags          → ["js","react"] (safe)

limitations:
  cloned.fn               → undefined
  typeof cloned.date      → "string"
```

---

### 6. Circular Reference Error

**Theory**: A circular reference is when an object refers back to itself (directly or indirectly). `JSON.stringify` cannot handle this and throws a `TypeError`. Solutions: use `structuredClone()`, manually remove circular refs, or use a library like `flatted`.

**Example**
```javascript
const obj = { name: "Alice" };
obj.self = obj;  // circular reference!

try {
  JSON.stringify(obj);
} catch (e) {
  console.log(e.message); // "Converting circular structure to JSON"
}

// Fix 1: structuredClone (modern, handles circular refs, no stringify needed)
const original = { a: 1, b: { c: 2 } };
const clone = structuredClone(original);   // ES2022
clone.b.c = 99;
console.log(original.b.c); // 2  (safe)

// Fix 2: custom replacer to skip circular keys
function safeStringify(obj) {
  const seen = new WeakSet();
  return JSON.stringify(obj, (key, value) => {
    if (typeof value === "object" && value !== null) {
      if (seen.has(value)) return "[Circular]";
      seen.add(value);
    }
    return value;
  });
}
const circ = { a: 1 };
circ.me = circ;
console.log(safeStringify(circ)); // '{"a":1,"me":"[Circular]"}'
```

**Output**
```
JSON.stringify(circular)    → TypeError: Converting circular structure
structuredClone(original)   → safe deep clone
safeStringify(circ)         → '{"a":1,"me":"[Circular]"}'
```

---

### Real-World Patterns

**Fetching and Parsing API Response**
```javascript
async function getUsers() {
  const response = await fetch("https://jsonplaceholder.typicode.com/users");

  if (!response.ok) throw new Error(`HTTP ${response.status}`);

  const users = await response.json(); // internally does JSON.parse
  return users.map(u => ({ id: u.id, name: u.name, email: u.email }));
}

getUsers().then(users => console.log(users[0]));
// { id: 1, name: "Leanne Graham", email: "..." }
```

**localStorage with JSON**
```javascript
// Save complex object to localStorage
const settings = { theme: "dark", language: "en", notifications: true };
localStorage.setItem("settings", JSON.stringify(settings));

// Read back
const saved = JSON.parse(localStorage.getItem("settings") || "{}");
console.log(saved.theme); // "dark"

// Helper functions
const storage = {
  set(key, val)  { localStorage.setItem(key, JSON.stringify(val)); },
  get(key, def)  { return JSON.parse(localStorage.getItem(key) ?? JSON.stringify(def)); },
  remove(key)    { localStorage.removeItem(key); },
};

storage.set("cart", [{ id: 1, qty: 2 }, { id: 5, qty: 1 }]);
const cart = storage.get("cart", []);
console.log(cart[0]); // { id: 1, qty: 2 }
```

**Output**
```
getUsers()[0]    → { id: 1, name: "Leanne Graham", email: "..." }
saved.theme      → "dark"
cart[0]          → { id: 1, qty: 2 }
```

---

[View Interview Questions](./interview.md)
