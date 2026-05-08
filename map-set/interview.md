# Map & Set — Interview Questions

---

**1. What is the main difference between a Map and a plain Object?**

| Feature | Object | Map |
|---|---|---|
| Key types | String / Symbol only | Any type (object, number, function) |
| Key order | Not guaranteed for numeric keys | Always insertion order |
| Size | Manual `Object.keys(o).length` | `.size` property |
| Iteration | Need `Object.entries()` | Directly iterable with `for...of` |

```javascript
const map = new Map();
const key = { id: 1 };
map.set(key, "user data");
console.log(map.get(key));  // "user data" — object as key works!

const obj = {};
// obj[key] → obj["[object Object]"] — key forced to string
```

---

**2. How do you remove duplicate values from an array using Set?**

Convert the array to a `Set` (which drops duplicates automatically), then spread it back into an array.

```javascript
const arr    = [1, 2, 2, 3, 3, 3, 4];
const unique = [...new Set(arr)];
console.log(unique); // [1, 2, 3, 4]

// Works with strings too
const tags = ["js", "css", "js", "html"];
console.log([...new Set(tags)]); // ["js","css","html"]
```

Note: this only deduplicates **primitives**. Objects are compared by reference, so two objects with the same content are treated as different.

---

**3. Does a Set preserve insertion order?**

Yes. A `Set` iterates in the order values were first added. Duplicate insertions do not change the position of the original value.

```javascript
const s = new Set();
s.add("c").add("a").add("b").add("a"); // "a" duplicate ignored
console.log([...s]); // ["c", "a", "b"] — insertion order
```

---

**4. What are WeakMap and WeakSet, and when would you use them?**

`WeakMap` and `WeakSet` hold **weak references** — if the key object (WeakMap) or value object (WeakSet) is garbage-collected elsewhere, the entry is automatically removed. They are not iterable and have no `.size`.

Use cases:
- **WeakMap**: attaching private metadata to objects without memory leaks
- **WeakSet**: tracking whether an object has been processed (like memoization, cycle detection)

```javascript
const meta = new WeakMap();

function attach(obj, info) { meta.set(obj, info); }
function getInfo(obj)      { return meta.get(obj); }

let user = { name: "Alice" };
attach(user, { role: "admin" });
console.log(getInfo(user)); // { role: "admin" }

user = null;  // object GC'd → WeakMap entry auto-cleaned → no memory leak
```

---

**5. What methods does Map provide for CRUD operations?**

```javascript
const m = new Map();

m.set("a", 1);          // Create / Update
console.log(m.get("a")); // Read → 1
console.log(m.has("a")); // Exists? → true
m.delete("a");           // Delete
console.log(m.has("a")); // → false
console.log(m.size);     // → 0
m.clear();               // Delete all
```

---

**6. How do you iterate over a Map? What are the three iteration methods?**

```javascript
const map = new Map([["x", 10], ["y", 20], ["z", 30]]);

// 1. keys()
for (const key of map.keys())   console.log(key);   // x, y, z

// 2. values()
for (const val of map.values()) console.log(val);   // 10, 20, 30

// 3. entries() [key, value]
for (const [key, val] of map.entries()) {
  console.log(`${key}=${val}`);  // x=10, y=20, z=30
}

// 4. forEach
map.forEach((val, key) => console.log(key, val));
// Note: forEach callback is (value, key) — opposite of entries!
```

---

**7. Can you convert between a Map and a plain Object?**

```javascript
// Object → Map
const obj = { a: 1, b: 2, c: 3 };
const map = new Map(Object.entries(obj));
console.log(map.get("b")); // 2

// Map → Object
const backToObj = Object.fromEntries(map);
console.log(backToObj); // { a: 1, b: 2, c: 3 }

// Map → Array of entries
const arr = [...map]; // [["a",1],["b",2],["c",3]]
```

---

**8. How does Set handle equality when checking for duplicates?**

Set uses **SameValueZero** comparison (similar to `===`, but `NaN === NaN` is true in a Set).

```javascript
const s = new Set();

s.add(NaN);
s.add(NaN);      // ignored — NaN equals NaN in Set
console.log(s.size); // 1

s.add(0);
s.add(-0);       // -0 and 0 are same in SameValueZero
console.log(s.size); // 2

s.add({});
s.add({});       // two different objects — both added
console.log(s.size); // 4
```

---

**9. Real-world: How would you count word frequency using a Map?**

```javascript
function countFreq(words) {
  const freq = new Map();
  for (const word of words) {
    freq.set(word, (freq.get(word) || 0) + 1);
  }
  return freq;
}

const words = ["apple","banana","apple","cherry","banana","apple"];
const freq  = countFreq(words);

console.log(freq.get("apple"));  // 3
console.log(freq.get("banana")); // 2

// Sort by frequency
const sorted = [...freq.entries()].sort((a, b) => b[1] - a[1]);
console.log(sorted); // [["apple",3],["banana",2],["cherry",1]]
```

---

**10. What is the time complexity of Map and Set operations compared to Array?**

| Operation | Array | Map / Set |
|---|---|---|
| Lookup (`has`) | O(n) linear scan | O(1) hash-based |
| Insert | O(1) push (amortized) | O(1) |
| Delete | O(n) splice + shift | O(1) |
| Unique check | O(n) includes | O(1) has |

```javascript
// Array: slow for large datasets
const arr = Array.from({length: 100000}, (_, i) => i);
console.time("array-includes");
arr.includes(99999); // O(n)
console.timeEnd("array-includes");

// Set: fast
const set = new Set(arr);
console.time("set-has");
set.has(99999);       // O(1)
console.timeEnd("set-has");
// set-has is ~100x faster on large datasets
```
