- Category: JavaScript
- Difficulty: Beginner
- Related: es6-features

### What are Array Methods?
Arrays in JavaScript come with built-in functions (methods) that allow you to easily add, remove, search, and transform data without writing manual `for` loops.

---

### 1. Adding and Removing Elements
**Theory**: These methods modify the original array by pushing/popping items from the end, or shifting/unshifting items from the beginning.

**Working Flow**
```text
[unshift] -> [ 1, 2, 3 ] <- [push]
 [shift]  <- [ 1, 2, 3 ] ->  [pop]
```

**Methods & Step-by-Step Example**:
- `push()`: Adds to the **end**.
- `pop()`: Removes from the **end**.
- `unshift()`: Adds to the **beginning**.
- `shift()`: Removes from the **beginning**.

```javascript
let queue = ["B", "C"];

// Step 1: Add to the end
queue.push("D"); 
// Result: ["B", "C", "D"]

// Step 2: Add to the beginning
queue.unshift("A"); 
// Result: ["A", "B", "C", "D"]

// Step 3: Remove from the end
let last = queue.pop(); 
// Result: ["A", "B", "C"] (last = "D")

// Step 4: Remove from the beginning
let first = queue.shift(); 
// Result: ["B", "C"] (first = "A")
```

---

### 2. Transforming Data (map)
**Theory**: `map()` is used when you want to transform *every* element in an array. It creates a brand **new array** of the exact same length.

**Working Flow**
```text
[ 1, 2, 3 ] --( x * 2 )--> [ 2, 4, 6 ]
```

**Step-by-Step Example**:
```javascript
const prices = [10, 20, 30];

// Step 1: Call map and pass a callback function
// Step 2: For each 'price', multiply it by 2
const doubled = prices.map(function(price) {
  return price * 2;
});

console.log(doubled); // [20, 40, 60]
console.log(prices);  // [10, 20, 30] (Original is unchanged!)
```

---

### 3. Filtering Data (filter)
**Theory**: `filter()` is used to remove elements that don't match a certain condition. It returns a **new array** containing only the items that passed the test (returned `true`).

**Working Flow**
```text
[ 1, 4, 5, 8 ] --( is Even? )--> [ 4, 8 ]
```

**Step-by-Step Example**:
```javascript
const ages = [12, 18, 25, 15];

// Step 1: Call filter and provide a test condition
// Step 2: Keep the item ONLY if the condition returns 'true'
const adults = ages.filter(age => {
  return age >= 18; 
});

console.log(adults); // [18, 25]
```

---

### 4. Accumulating Data (reduce)
**Theory**: `reduce()` is used to boil down an entire array into a **single value** (like a total sum, or a combined string).

**Working Flow**
```text
[ 1, 2, 3 ] --( Add to Total )--> 6
```

**Step-by-Step Example**:
```javascript
const expenses = [50, 100, 50];

// Step 1: Provide an accumulator ('total') and current item ('cost')
// Step 2: Set the initial value of 'total' to 0 (the second argument)
// Step 3: Loop through and keep adding to 'total'
const sum = expenses.reduce((total, cost) => {
  return total + cost;
}, 0); 

console.log(sum); // 200
```

---

### 5. Searching Elements (find & includes)
**Theory**: Use these when you need to know if an item exists, or grab the very first item that matches a condition.

**Key Methods**:
- `includes()`: Returns `true` or `false` if the exact value exists.
- `find()`: Returns the **first item** (usually an object) that matches a condition, or `undefined` if not found.

**Step-by-Step Example**:
```javascript
const fruits = ["Apple", "Mango", "Banana"];

// 1. Using includes() for simple arrays
const hasMango = fruits.includes("Mango"); 
console.log(hasMango); // true

// 2. Using find() for complex searches
const users = [
  { id: 1, name: "Richa" },
  { id: 2, name: "John" }
];

const foundUser = users.find(user => user.id === 1);
console.log(foundUser.name); // "Richa"
```

---

[View Interview Questions](./interview.md)
