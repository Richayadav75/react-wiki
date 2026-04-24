- Category: JavaScript
- Difficulty: Beginner
- Related: data-types

### What are Object Methods?
Objects are key-value pairs. JavaScript provides built-in `Object` methods to extract keys, values, or merge objects together.

---

### 1. Extracting Keys and Values
**Theory**: You can break an object down into arrays of just its keys, just its values, or both (entries).

**Working Flow**
```text
{ a: 1, b: 2 } --> Object.keys() --> ["a", "b"]
{ a: 1, b: 2 } --> Object.values() --> [1, 2]
```

**Key Methods**:
- `Object.keys(obj)`: Returns an array of property names.
- `Object.values(obj)`: Returns an array of property values.
- `Object.entries(obj)`: Returns an array of [key, value] pairs.

**Step-by-Step Example**:
```javascript
const user = { name: "Richa", role: "Admin", age: 25 };

// Step 1: Get all the keys
const keys = Object.keys(user); 
console.log(keys); // ["name", "role", "age"]

// Step 2: Get all the values
const values = Object.values(user);
console.log(values); // ["Richa", "Admin", 25]

// Step 3: Loop through keys and values together
Object.entries(user).forEach(([key, value]) => {
  console.log(`${key} is ${value}`);
});
```

---

### 2. Merging and Cloning Objects
**Theory**: You often need to combine objects or create a copy of an object without mutating the original.

**Key Method**:
- `Object.assign(target, ...sources)`: Copies properties from source objects to a target object. (Note: Spread syntax `{...obj}` is now more common).

**Step-by-Step Example**:
```javascript
const target = { a: 1, b: 2 };
const source = { b: 4, c: 5 };

// Step 1: Merge source into target (overwrites existing keys)
const returnedTarget = Object.assign(target, source);

console.log(target); // { a: 1, b: 4, c: 5 }
```

---

[View Interview Questions](./interview.md)
