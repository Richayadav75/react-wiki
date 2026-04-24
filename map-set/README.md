- Category: JavaScript
- Difficulty: Intermediate
- Related: object-methods

### What are Map and Set?
Introduced in ES6, `Map` and `Set` are advanced data structures that solve common limitations found in traditional Arrays and Objects.

---

### 1. Map (Key-Value Pairs)
**Theory**: Like Objects, Maps hold key-value pairs. But unlike Objects, Map keys can be *any* data type (even other objects or functions), and they remember the exact order of insertion.

**Key Features**:
- **set(k, v)**: Adds a pair.
- **get(k)**: Retrieves a value.
- **size**: Returns the number of elements.

**Step-by-Step Example**:
```javascript
// Step 1: Create a Map
let userRoles = new Map();

// Step 2: Add data (Keys can be anything!)
userRoles.set("Richa", "Admin");
userRoles.set(123, "Guest"); 

// Step 3: Read data
console.log(userRoles.get("Richa")); // "Admin"
console.log(userRoles.size);         // 2
```

---

### 2. Set (Unique Values)
**Theory**: A Set is a collection of values where each value must be **unique**. If you try to add a duplicate, it is simply ignored.

**Step-by-Step Example**:
```javascript
// Step 1: Create a Set from an array with duplicates
const numbers = new Set([1, 2, 2, 3, 3, 3]);

// Step 2: The duplicates are automatically removed!
console.log(numbers); // Set { 1, 2, 3 }

// Step 3: Add new values
numbers.add(4);
numbers.add(1); // Ignored, 1 already exists
```

---

[View Interview Questions](./interview.md)
