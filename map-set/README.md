- Category: JavaScript
- Difficulty: Intermediate
- Related: object-methods, array-methods, es6-features

### Map & Set — Specialized Collections
ES6 introduced two powerful data structures: **Map** (key-value pairs with any key type) and **Set** (unique value collections). They are purpose-built for tasks where plain objects and arrays fall short — and they are significantly more efficient for frequent lookups, insertions, and deletions.

**Analogy**
**Map** = a smart address book where the "name" can be anything — a string, a number, even another object. **Set** = a guest list where each name appears only once, no matter how many times you try to add the same person.

---

### 1. Map — Key-Value with Any Key Type

**Theory**: A `Map` stores key-value pairs like a plain object, but with three critical advantages: keys can be **any type** (objects, functions, numbers), insertion **order is preserved**, and it has a built-in `.size` property.

**Working Flow**
![flow-chart](flow-chart.png)

**Example**
```javascript
const map = new Map();

// set — add entries
map.set("city",  "Mumbai");
map.set(1,       "one");
map.set(true,    "boolean key");

const keyObj = { id: 99 };
map.set(keyObj, "object as key");

// get — retrieve
console.log(map.get("city"));   // "Mumbai"
console.log(map.get(1));        // "one"
console.log(map.get(true));     // "boolean key"
console.log(map.get(keyObj));   // "object as key"
console.log(map.get("missing")); // undefined

// has — existence check
console.log(map.has("city"));   // true
console.log(map.has("name"));   // false

// size
console.log(map.size);          // 4

// delete
map.delete(1);
console.log(map.size);          // 3

// clear
// map.clear(); → empties the map
```

**Output**
```
map.get("city")    → "Mumbai"
map.get(1)         → "one"
map.get(true)      → "boolean key"
map.get(keyObj)    → "object as key"
map.get("missing") → undefined
map.has("city")    → true
map.has("name")    → false
map.size           → 4
after delete(1): size → 3
```

---

### 2. Map vs Object — When to Use Which

**Theory**: Both store key-value data, but they are optimized for different use cases. Use `Map` when keys are non-strings, when you need reliable insertion order, or when you are frequently adding/removing entries.

**Working Flow**
![flow-chart-2](flow-chart-2.png)

| Feature | Plain Object | Map |
|---|---|---|
| Key types | String / Symbol only | Any type |
| Key order | Not guaranteed (numbers sort first) | Insertion order always |
| Size | `Object.keys(obj).length` | `map.size` |
| Iterable | No (need Object.keys etc.) | Yes — `for...of` directly |
| Performance (add/delete) | OK | Better for frequent ops |
| JSON serializable | Yes | No (need manual conversion) |

**Example**
```javascript
// Object: number keys get sorted
const obj = {};
obj[3] = "three";
obj[1] = "one";
obj[2] = "two";
console.log(Object.keys(obj)); // ["1","2","3"] — sorted by number!

// Map: preserves insertion order
const map = new Map();
map.set(3, "three");
map.set(1, "one");
map.set(2, "two");
console.log([...map.keys()]); // [3, 1, 2] — insertion order
```

**Output**
```
Object.keys(obj) → ["1","2","3"]  (sorted)
[...map.keys()]  → [3, 1, 2]     (insertion order preserved)
```

---

### 3. Iterating a Map

**Theory**: Maps are directly iterable with `for...of`. You can iterate over keys, values, or entries (key-value pairs). This is cleaner than the object equivalents (`Object.keys`, `Object.values`).

**Example**
```javascript
const scores = new Map([
  ["Alice", 95],
  ["Bob",   82],
  ["Cara",  91],
]);

// keys
for (const name of scores.keys()) {
  console.log(name);   // Alice, Bob, Cara
}

// values
for (const score of scores.values()) {
  console.log(score);  // 95, 82, 91
}

// entries [key, value]
for (const [name, score] of scores.entries()) {
  console.log(`${name}: ${score}`);
}
// Alice: 95
// Bob: 82
// Cara: 91

// forEach
scores.forEach((score, name) => {
  console.log(`${name} scored ${score}`);
});

// spread to array
const arr = [...scores]; // [["Alice",95], ["Bob",82], ["Cara",91]]
```

**Output**
```
keys:    Alice → Bob → Cara
values:  95 → 82 → 91
entries: Alice: 95 / Bob: 82 / Cara: 91
spread:  [["Alice",95],["Bob",82],["Cara",91]]
```

---

### 4. Set — Unique Value Collections

**Theory**: A `Set` stores **unique values** of any type. Adding a value that already exists is silently ignored. Sets have add/has/delete/size just like Map, and they preserve insertion order.

**Working Flow**
![flow-chart-3](flow-chart-3.png)

**Example**
```javascript
const set = new Set();

set.add("apple");
set.add("banana");
set.add("apple");  // duplicate — ignored
set.add("mango");
set.add("banana"); // duplicate — ignored

console.log(set);         // Set { "apple", "banana", "mango" }
console.log(set.size);    // 3

console.log(set.has("apple"));  // true
console.log(set.has("grape"));  // false

set.delete("banana");
console.log(set.size);   // 2

// Iterating
for (const fruit of set) {
  console.log(fruit);    // apple, mango
}

// Spread to array
const arr = [...set];   // ["apple", "mango"]
```

**Output**
```
set after adds       → Set { "apple", "banana", "mango" }
set.size             → 3
set.has("apple")     → true
set.has("grape")     → false
after delete:size    → 2
spread               → ["apple","mango"]
```

---

### 5. Set for Array Deduplication

**Theory**: The most common real-world use of `Set` is removing duplicates from an array in one line. Converting an array to a Set drops all duplicates; spreading it back gives a clean unique array.

**Working Flow**
![flow-chart-4](flow-chart-4.png)

**Example**
```javascript
// Numbers
const nums    = [1, 2, 2, 3, 3, 3, 4];
const unique  = [...new Set(nums)];
console.log(unique);           // [1, 2, 3, 4]

// Strings
const tags    = ["js", "css", "js", "html", "css", "js"];
const uniqTags = [...new Set(tags)];
console.log(uniqTags);         // ["js","css","html"]

// Count unique items
console.log(new Set(tags).size); // 3

// Remove duplicate objects — note: only works for primitives!
// Objects are compared by reference, not by value
const users = [{id:1},{id:2},{id:1}];
console.log([...new Set(users)].length); // 3 — not deduplicated!

// Correct way to deduplicate objects by id:
const uniqueUsers = [...new Map(users.map(u => [u.id, u])).values()];
console.log(uniqueUsers); // [{id:1},{id:2}]
```

**Output**
```
unique nums      → [1, 2, 3, 4]
unique tags      → ["js","css","html"]
unique tag count → 3
dedup objects (wrong) → 3 items (refs differ)
dedup by id    → [{id:1},{id:2}]
```

---

### 6. WeakMap & WeakSet

**Theory**: `WeakMap` and `WeakSet` are "weak" versions — they hold **weak references** to their keys/values. If the object used as a key is garbage-collected, the entry disappears automatically. They are not iterable and have no `.size`. Use them for **private data** and **caching** without memory leaks.

**Working Flow**
![flow-chart-5](flow-chart-5.png)

**Example**
```javascript
// WeakMap: attach private data to objects
const cache = new WeakMap();

function process(obj) {
  if (cache.has(obj)) {
    console.log("From cache:", cache.get(obj));
    return cache.get(obj);
  }
  const result = obj.value * 2;  // expensive computation
  cache.set(obj, result);
  return result;
}

let data = { value: 21 };
console.log(process(data)); // 42 (computed)
console.log(process(data)); // 42 (from cache)

data = null;  // original object eligible for GC
// cache entry is automatically cleaned up — no memory leak!

// WeakSet: track objects without preventing GC
const seen = new WeakSet();
function firstVisit(obj) {
  if (seen.has(obj)) return false;
  seen.add(obj);
  return true;
}

const user = { name: "Alice" };
console.log(firstVisit(user)); // true
console.log(firstVisit(user)); // false
```

**Output**
```
process(data) first  → 42 (computed)
process(data) second → 42 (from cache)
firstVisit(user) #1  → true
firstVisit(user) #2  → false
```

---

### Real-World Patterns

**Word Frequency Counter with Map**
```javascript
function wordFrequency(text) {
  const freq = new Map();
  const words = text.toLowerCase().split(/\W+/).filter(Boolean);
  for (const word of words) {
    freq.set(word, (freq.get(word) || 0) + 1);
  }
  return freq;
}

const text = "the cat sat on the mat the cat";
const freq  = wordFrequency(text);

console.log(freq.get("the")); // 3
console.log(freq.get("cat")); // 2

// Top word
const top = [...freq.entries()].sort((a,b) => b[1]-a[1])[0];
console.log(`"${top[0]}" appears ${top[1]} times`); // "the" appears 3 times
```

**Output**
```
freq.get("the")  → 3
freq.get("cat")  → 2
top word         → "the" appears 3 times
```

---

[View Interview Questions](./interview.md)
