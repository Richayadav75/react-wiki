# Object Methods — Interview Questions

---

**1. What do `Object.keys()`, `Object.values()`, and `Object.entries()` return?**

All three operate on the object's **own enumerable** properties and return arrays:

```javascript
const user = { name: "Alice", age: 28, role: "admin" };

Object.keys(user);    // → ["name","age","role"]
Object.values(user);  // → ["Alice",28,"admin"]
Object.entries(user); // → [["name","Alice"],["age",28],["role","admin"]]

// Common usage — iterate with both key and value
Object.entries(user).forEach(([key, val]) => {
  console.log(`${key}: ${val}`);
});
```

---

**2. What is the difference between `Object.freeze()` and `Object.seal()`?**

| Operation | `seal()` | `freeze()` |
|-----------|----------|------------|
| Add property | No | No |
| Delete property | No | No |
| Update value | Yes | No |

```javascript
const sealed = Object.seal({ x: 1 });
sealed.x = 99;    // ✓ allowed
sealed.y = 2;     // ✗ silently fails
delete sealed.x;  // ✗ silently fails
console.log(sealed); // → { x: 99 }

const frozen = Object.freeze({ x: 1 });
frozen.x = 99;    // ✗ silently fails (TypeError in strict)
console.log(frozen); // → { x: 1 }
```

Both are **shallow** — nested objects inside frozen/sealed objects are still mutable.

---

**3. What is the difference between `Object.assign()` and the spread operator for merging objects?**

Both merge objects but behave differently:

```javascript
const a = { x: 1 };
const b = { y: 2 };

// Object.assign — MUTATES the first argument
const result1 = Object.assign(a, b);
console.log(a);       // → { x:1, y:2 }  (a was mutated!)
console.log(result1 === a); // → true

// Spread — creates a NEW object (preferred)
const result2 = { ...a, ...b };
console.log(result2 === a); // → false  (new object)
```

Always use `Object.assign({}, source1, source2)` (with empty target) or spread to avoid mutating.

---

**4. What is a shallow copy and when does it cause problems?**

A shallow copy duplicates only the top-level properties. Nested objects are still referenced (shared).

```javascript
const original = { name: "Alice", address: { city: "Mumbai" } };
const copy = { ...original };

copy.name = "Bob";           // ✓ independent (primitive)
copy.address.city = "Delhi"; // ✗ BOTH objects affected! (shared reference)

console.log(original.address.city); // → "Delhi" (mutated!)
```

For deep copies use `structuredClone(obj)` (modern) or `JSON.parse(JSON.stringify(obj))` (simple but loses functions/dates).

---

**5. How do you check if a property belongs to an object itself vs being inherited?**

```javascript
const base = { type: "animal" };
const dog  = Object.create(base);
dog.name   = "Rex";

// hasOwnProperty — own only
dog.hasOwnProperty("name");  // → true
dog.hasOwnProperty("type");  // → false (inherited)

// in operator — includes inherited
"name" in dog;  // → true
"type" in dog;  // → true (inherited, still in chain)

// Modern (ES2022)
Object.hasOwn(dog, "name"); // → true
Object.hasOwn(dog, "type"); // → false
```

Use `hasOwnProperty` (or `Object.hasOwn`) when you only care about own properties. Use `in` when the prototype chain matters.

---

**6. What does `Object.fromEntries()` do and how is it used with `Object.entries()`?**

`Object.fromEntries()` converts an array of `[key, value]` pairs back into an object. It is the inverse of `Object.entries()`, enabling a map-filter-transform pipeline on objects.

```javascript
const prices = { apple: 50, banana: 30, mango: 80 };

// Apply 10% discount to all prices
const discounted = Object.fromEntries(
  Object.entries(prices).map(([item, price]) => [item, price * 0.9])
);
console.log(discounted);
// → { apple:45, banana:27, mango:72 }

// Filter out expensive items
const cheap = Object.fromEntries(
  Object.entries(prices).filter(([_, price]) => price < 60)
);
console.log(cheap);
// → { apple:50, banana:30 }
```

---

**7. What is the `for...in` loop and what is its main danger?**

`for...in` iterates over all **enumerable** properties of an object, including inherited ones from the prototype chain.

```javascript
const base = { inherited: true };
const obj  = Object.create(base);
obj.own1 = "a";
obj.own2 = "b";

for (const key in obj) {
  console.log(key); // → "own1", "own2", "inherited"  (inherited included!)
}

// Safe pattern — guard with hasOwnProperty
for (const key in obj) {
  if (Object.hasOwn(obj, key)) {
    console.log(key); // → "own1", "own2" only
  }
}

// Modern alternative — use Object.keys() which only returns own properties
Object.keys(obj).forEach(key => console.log(key)); // → "own1", "own2"
```

---

**8. What are computed property names and when do you use them?**

Computed property names use `[expression]` as keys in object literals. Useful for dynamic/data-driven object construction.

```javascript
const field = "email";
const user = {
  [field]: "alice@example.com",          // → user.email
  [`${field}_verified`]: true,           // → user.email_verified
};

// Common use: action handlers (Redux pattern)
function createReducer(initialState, handlers) {
  return (state = initialState, action) =>
    handlers[action.type]
      ? handlers[action.type](state, action)
      : state;
}

const reducer = createReducer(0, {
  INCREMENT: (state) => state + 1,
  DECREMENT: (state) => state - 1,
});
```

---

**9. How do you transform all keys of an object from snake_case to camelCase?**

```javascript
function toCamelCase(str) {
  return str.replace(/_([a-z])/g, (_, c) => c.toUpperCase());
}

function transformKeys(obj) {
  return Object.fromEntries(
    Object.entries(obj).map(([k, v]) => [toCamelCase(k), v])
  );
}

const api = { user_name: "alice", created_at: "2024-01-01", is_active: true };
console.log(transformKeys(api));
// → { userName:"alice", createdAt:"2024-01-01", isActive:true }
```

---

**10. What does `Object.create(null)` do and why would you use it?**

`Object.create(null)` creates an object with **no prototype at all** — a pure key-value dictionary without `toString`, `hasOwnProperty`, or any other inherited methods.

```javascript
const dict = Object.create(null);
dict["name"] = "Alice";
dict["toString"] = "custom"; // safe — no conflict with Object.prototype

console.log(dict.__proto__); // → undefined
console.log(Object.getPrototypeOf(dict)); // → null

// Use case: caches, lookup tables — prevents prototype pollution attacks
// where an attacker might set __proto__ or constructor as a key
```
