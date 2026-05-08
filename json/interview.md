# JSON — Interview Questions

---

**1. What does JSON stand for, and what is it used for?**

JSON stands for **JavaScript Object Notation**. It is a text-based, language-independent data format used to exchange structured data between a client (browser) and a server, and to store configuration or state (e.g., in localStorage or config files). Every major language can parse it.

---

**2. What is the difference between `JSON.stringify()` and `JSON.parse()`?**

- `JSON.stringify(obj)` — converts a JavaScript object/value into a JSON **string** (for sending over the network or saving to storage).
- `JSON.parse(str)` — converts a JSON **string** back into a JavaScript object (for reading and working with the data).

```javascript
const obj = { name: "Alice", age: 30 };

const str = JSON.stringify(obj);
console.log(str);           // '{"name":"Alice","age":30}'
console.log(typeof str);    // "string"

const back = JSON.parse(str);
console.log(back.name);     // "Alice"
console.log(typeof back);   // "object"
```

---

**3. What types are NOT supported in JSON, and what happens to them during stringify?**

| Type | In object property | In array |
|---|---|---|
| `undefined` | Key is dropped silently | Becomes `null` |
| `function` | Key is dropped silently | Becomes `null` |
| `Symbol` | Key is dropped silently | Becomes `null` |
| `NaN` / `Infinity` | Becomes `null` | Becomes `null` |
| `Date` | Converted to ISO string | ISO string |
| `BigInt` | Throws `TypeError` | Throws |

```javascript
const data = { a: undefined, b: () => {}, c: NaN, d: new Date("2024-01-01") };
console.log(JSON.stringify(data));
// '{"c":null,"d":"2024-01-01T00:00:00.000Z"}'
// a and b are dropped entirely
```

---

**4. How do you use the `replacer` parameter in `JSON.stringify`?**

The second argument to `JSON.stringify` can be an **array** (whitelist of keys to include) or a **function** (transform or exclude values).

```javascript
const user = { name: "Alice", password: "secret", age: 30 };

// Array replacer — only include specific keys
console.log(JSON.stringify(user, ["name", "age"]));
// '{"name":"Alice","age":30}'

// Function replacer — exclude sensitive fields
const safe = JSON.stringify(user, (key, value) => {
  if (key === "password") return undefined; // exclude
  return value;
});
console.log(safe); // '{"name":"Alice","age":30}'
```

---

**5. What does the `reviver` parameter in `JSON.parse` do?**

The `reviver` is a function that transforms each key-value pair **after** parsing. Commonly used to restore `Date` strings back into real `Date` objects.

```javascript
const json = '{"name":"Alice","createdAt":"2024-01-15T00:00:00.000Z"}';

const obj = JSON.parse(json, (key, value) => {
  if (key === "createdAt") return new Date(value);
  return value;
});

console.log(obj.createdAt instanceof Date);  // true
console.log(obj.createdAt.getFullYear());    // 2024
```

---

**6. How can you deep-clone an object using JSON? What are the limitations?**

```javascript
const original = { user: "Alice", prefs: { theme: "dark" } };
const clone    = JSON.parse(JSON.stringify(original));

clone.prefs.theme = "light";
console.log(original.prefs.theme); // "dark" — clone is independent
```

**Limitations:**
- Functions and `undefined` are dropped
- `Date` objects become strings (not restored to `Date`)
- Fails with circular references (throws `TypeError`)
- Does not copy non-enumerable properties or the prototype chain

**Better alternatives**: `structuredClone()` (handles more types + circular refs) or Lodash `_.cloneDeep()`.

---

**7. What happens when you `JSON.stringify` an object with a circular reference?**

It throws a `TypeError: Converting circular structure to JSON`.

```javascript
const obj = {};
obj.self = obj;  // circular

try {
  JSON.stringify(obj);
} catch(e) {
  console.log(e.message); // "Converting circular structure to JSON"
}

// Fix: structuredClone handles circular refs safely
// Or use a custom replacer with a WeakSet to track seen objects
```

---

**8. Why must keys be double-quoted in JSON? Are comments allowed?**

JSON follows the RFC 8259 standard designed for language-interoperability. All languages' parsers expect **double-quoted strings** as keys. Single quotes or unquoted keys are JavaScript-only syntax. Comments are **not allowed** in standard JSON — they would require every parser in every language to support them.

```javascript
// Valid JSON:
const valid = '{"name":"Alice","age":30}';

// Invalid JSON (throws SyntaxError):
// {"name": 'Alice'}    — single quotes
// {name: "Alice"}      — unquoted key
// {"name":"Alice"}// comment  — comments
JSON.parse(valid);  // OK
JSON.parse("{name: 'Alice'}"); // SyntaxError
```

---

**9. How do you safely read from localStorage using JSON, handling missing or corrupt data?**

```javascript
function readStorage(key, defaultValue) {
  try {
    const raw = localStorage.getItem(key);
    if (raw === null) return defaultValue;  // key doesn't exist
    return JSON.parse(raw);
  } catch (e) {
    console.warn(`Corrupt data for key "${key}":`, e.message);
    return defaultValue;
  }
}

// Usage
const cart = readStorage("cart", []);
const user = readStorage("user", null);
```

---

**10. What is the `space` parameter in `JSON.stringify` and when is it useful?**

The third argument to `JSON.stringify` controls indentation for **pretty-printing**. Pass a number (spaces) or a string (used as indent). Useful for debugging, log files, or saving human-readable config files.

```javascript
const data = { name: "Alice", scores: [95, 88, 72] };

// Compact (default)
JSON.stringify(data);
// '{"name":"Alice","scores":[95,88,72]}'

// Pretty with 2 spaces
JSON.stringify(data, null, 2);
// {
//   "name": "Alice",
//   "scores": [
//     95,
//     88,
//     72
//   ]
// }

// Custom indent string
JSON.stringify(data, null, "\t"); // tab-indented
```
