- Category: JavaScript
- Difficulty: Beginner
- Related: data-types, es6-features, prototypes, array-methods

### JavaScript Object Methods — Working with Key-Value Data
Objects are the backbone of JavaScript — every React component's props, every API response, every config is an object. The global `Object` constructor provides powerful static methods to inspect, copy, protect, and transform these structures.

**Analogy**
An object is like a filing cabinet with labelled drawers. `Object.keys()` gives you a list of drawer labels. `Object.values()` gives you the contents. `Object.entries()` gives you both together. `Object.freeze()` puts a padlock on the whole cabinet.

---

### 1. Object.keys / Object.values / Object.entries
**Theory**: Three essential methods for iterating over an object's data. They all return arrays, making it easy to use `.map()`, `.filter()`, and `.reduce()` on object data.

**Working Flow**
![flow-chart](flow-chart.png)

**Example**
```javascript
const product = {
  id:    101,
  name:  "Laptop",
  price: 75000,
  stock: 12
};

// keys
const keys = Object.keys(product);
console.log(keys); // → ["id","name","price","stock"]

// values
const values = Object.values(product);
console.log(values); // → [101,"Laptop",75000,12]

// entries — use for iteration with both key and value
Object.entries(product).forEach(([key, val]) => {
  console.log(`${key}: ${val}`);
});
// → "id: 101"
// → "name: Laptop"
// → "price: 75000"
// → "stock: 12"

// Real use: transform values
const discounted = Object.fromEntries(
  Object.entries(product).map(([key, val]) =>
    key === "price" ? [key, val * 0.9] : [key, val]
  )
);
console.log(discounted.price); // → 67500  (10% off)

// Count properties
console.log(Object.keys(product).length); // → 4
```

**Output**
```
Object.keys(product)    → ["id","name","price","stock"]
Object.values(product)  → [101,"Laptop",75000,12]
entries forEach         → id: 101 / name: Laptop / price: 75000 / stock: 12
discounted.price        → 67500
Object.keys().length    → 4
```

---

### 2. Object.assign vs Spread — Cloning and Merging
**Theory**: Both copy properties from one or more source objects into a target. The spread operator (`{...obj}`) is the modern preferred approach. Both perform **shallow** copies — nested objects are NOT deep-cloned.

**Working Flow**
![flow-chart-2](flow-chart-2.png)

**Example**
```javascript
const defaults = { theme: "light", lang: "en", fontSize: 16 };
const userPrefs = { theme: "dark", fontSize: 20 };

// Object.assign — MUTATES first argument
const config1 = Object.assign({}, defaults, userPrefs);
console.log(config1);
// → { theme:"dark", lang:"en", fontSize:20 }

// Spread — creates new object (preferred)
const config2 = { ...defaults, ...userPrefs };
console.log(config2);
// → { theme:"dark", lang:"en", fontSize:20 }

// Shallow copy warning — nested objects are SHARED
const original = { name: "Alice", address: { city: "Mumbai" } };
const clone     = { ...original };

clone.name          = "Bob";   // ✓ independent
clone.address.city  = "Delhi"; // ✗ BOTH affected (shared reference)

console.log(original.name);         // → "Alice" (independent)
console.log(original.address.city); // → "Delhi" (shared — mutated!)

// Deep clone (when you need it)
const deepClone = JSON.parse(JSON.stringify(original)); // simple but lossy
// or: structuredClone(original);  // modern, handles more types
```

**Output**
```
config1                   → {theme:"dark",lang:"en",fontSize:20}
config2                   → {theme:"dark",lang:"en",fontSize:20}
original.name             → "Alice"    (shallow — independent)
original.address.city     → "Delhi"    (shallow — shared, mutated!)
```

**Explanation**: For top-level merges, spread is clean and sufficient. When you need to deeply clone nested objects, use `structuredClone()` or a library like Lodash.

---

### 3. Object.freeze / Object.seal — Protecting Objects
**Theory**: These methods restrict what can be done to an object after creation.
- `freeze()` — completely locked: no add, delete, or update.
- `seal()` — partially locked: can update existing values, but no add or delete.

**Working Flow**
![flow-chart-3](flow-chart-3.png)

**Example**
```javascript
// Object.freeze — immutable
const CONFIG = Object.freeze({
  API_URL:  "https://api.example.com",
  TIMEOUT:  5000,
  VERSION:  "2.1.0"
});

CONFIG.API_URL = "https://hack.com";  // silently fails (strict: TypeError)
CONFIG.newProp = "test";              // silently fails
delete CONFIG.VERSION;                // silently fails

console.log(CONFIG.API_URL); // → "https://api.example.com"  (unchanged)
console.log(Object.isFrozen(CONFIG)); // → true

// Object.seal — can update, cannot add or delete
const user = Object.seal({
  name: "Alice",
  role: "viewer"
});

user.name   = "Bob";    // ✓ update allowed
user.age    = 28;       // ✗ silently fails (can't add)
delete user.role;       // ✗ silently fails (can't delete)

console.log(user); // → { name:"Bob", role:"viewer" }
console.log(Object.isSealed(user)); // → true

// Freeze is SHALLOW — nested objects are still mutable
const state = Object.freeze({ user: { name: "Alice" } });
state.user.name = "Bob"; // ✓ Works! (freeze is shallow)
console.log(state.user.name); // → "Bob"
```

**Output**
```
CONFIG.API_URL (after freeze attempt) → "https://api.example.com"
Object.isFrozen(CONFIG)               → true
user after seal                       → {name:"Bob",role:"viewer"}
Object.isSealed(user)                 → true
state.user.name (after nested mutate) → "Bob"  (freeze is shallow!)
```

---

### 4. Object.create — Prototype-Based Creation
**Theory**: `Object.create(proto)` creates a new object with the given object as its prototype. The new object inherits all properties and methods from the prototype. Passing `null` creates an object with NO prototype (a pure hash map).

**Working Flow**
![flow-chart-4](flow-chart-4.png)

**Example**
```javascript
// Base "class" via prototype
const Vehicle = {
  type: "vehicle",
  describe() {
    return `${this.name} is a ${this.type} going ${this.speed}km/h`;
  }
};

const car = Object.create(Vehicle);
car.name  = "Tesla";
car.type  = "car";
car.speed = 120;

console.log(car.describe());
// → "Tesla is a car going 120km/h"
console.log(Object.getPrototypeOf(car) === Vehicle); // → true

// Pure dictionary — no prototype pollution risk
const dict = Object.create(null);
dict["key1"] = "value1";
dict["toString"] = "my-string"; // safe, no built-in toString to conflict
console.log(dict.key1);         // → "value1"
console.log(dict.__proto__);    // → undefined (no prototype!)
```

**Output**
```
car.describe()                       → "Tesla is a car going 120km/h"
Object.getPrototypeOf(car)===Vehicle → true
dict.key1                            → "value1"
dict.__proto__                       → undefined
```

---

### 5. hasOwnProperty and for...in Loop
**Theory**: `hasOwnProperty(key)` checks if a property belongs directly to the object (not inherited). The `for...in` loop iterates over ALL enumerable properties including inherited ones — so always pair it with `hasOwnProperty`.

**Working Flow**
![flow-chart-5](flow-chart-5.png)

**Example**
```javascript
const base = { type: "animal" };
const dog  = Object.create(base);
dog.name   = "Rex";
dog.breed  = "Labrador";

// for...in — includes inherited
for (const key in dog) {
  console.log(key); // → "name", "breed", "type"  (includes inherited!)
}

// for...in with hasOwnProperty — only own
for (const key in dog) {
  if (dog.hasOwnProperty(key)) {
    console.log(key); // → "name", "breed"  (own only)
  }
}

// Modern alternative: Object.keys (only own, enumerable)
Object.keys(dog); // → ["name", "breed"]

// hasOwnProperty check
dog.hasOwnProperty("name");  // → true  (own)
dog.hasOwnProperty("type");  // → false (inherited)

// Safer alternative (prototype-pollution safe)
Object.prototype.hasOwnProperty.call(dog, "name"); // → true
// or
Object.hasOwn(dog, "name"); // → true (ES2022)
```

**Output**
```
for...in (all)      → name, breed, type
for...in (own only) → name, breed
Object.keys(dog)    → ["name","breed"]
hasOwnProperty("name")  → true
hasOwnProperty("type")  → false
Object.hasOwn(dog,"name")→ true
```

---

### 6. Computed Property Names and Object Shorthand
**Theory**: ES6 allows you to use dynamic expressions as property keys using `[expression]`. Combined with shorthand properties and methods, this makes object construction much more flexible.

**Example**
```javascript
// Computed property names
const field = "username";
const user = {
  [field]: "alice",                  // dynamic key
  [`is_${field}_valid`]: true,       // template expression as key
};
console.log(user.username);          // → "alice"
console.log(user.is_username_valid); // → true

// Building objects from dynamic data
function makeAction(type, payload) {
  return { type, payload, timestamp: Date.now() };
}
makeAction("LOGIN", { userId: 1 });
// → { type:"LOGIN", payload:{userId:1}, timestamp:1700000000 }

// Dynamic key accumulation (common Redux-like pattern)
const initialState = {};
const actions = ["increment", "decrement", "reset"];

const handlers = actions.reduce((acc, action) => {
  return { ...acc, [action]: (state) => state };
}, {});
console.log(Object.keys(handlers));
// → ["increment","decrement","reset"]
```

**Output**
```
user.username          → "alice"
user.is_username_valid → true
makeAction(...)        → {type:"LOGIN",payload:{userId:1},timestamp:...}
Object.keys(handlers)  → ["increment","decrement","reset"]
```

---

### Real-World Patterns

**1. Merging API response with local defaults**
```javascript
const apiDefaults = { perPage: 20, sort: "created_at", order: "desc" };

function buildQueryParams(userOptions = {}) {
  const merged = { ...apiDefaults, ...userOptions };
  return Object.entries(merged)
    .map(([k, v]) => `${k}=${encodeURIComponent(v)}`)
    .join("&");
}

console.log(buildQueryParams({ perPage: 50, search: "react" }));
// → "perPage=50&sort=created_at&order=desc&search=react"
```

**2. Transforming API response keys (snake_case to camelCase)**
```javascript
function toCamelCase(str) {
  return str.replace(/_([a-z])/g, (_, c) => c.toUpperCase());
}

function transformKeys(obj) {
  return Object.fromEntries(
    Object.entries(obj).map(([k, v]) => [toCamelCase(k), v])
  );
}

const apiResponse = { user_name: "alice", created_at: "2024-01-01", is_active: true };
console.log(transformKeys(apiResponse));
// → { userName:"alice", createdAt:"2024-01-01", isActive:true }
```

**Output**
```
buildQueryParams(...)  → "perPage=50&sort=created_at&order=desc&search=react"
transformKeys(...)     → {userName:"alice",createdAt:"2024-01-01",isActive:true}
```

---

[View Interview Questions](./interview.md)
