- Category: JavaScript
- Difficulty: Beginner
- Related: Fundamentals

### What are Math & Number Methods?
JavaScript provides a built-in `Math` object for mathematical tasks and `Number` methods for formatting.

---

### 1. Rounding Numbers
**Theory**: You can round numbers to the nearest integer, forcefully round up, or forcefully round down.

**Working Flow**
```text
3.14 --( round )--> 3
3.14 --( ceil )---> 4
3.99 --( floor )--> 3
```

**Key Methods**:
- `Math.round(x)`: Rounds to nearest integer.
- `Math.ceil(x)`: Always rounds UP.
- `Math.floor(x)`: Always rounds DOWN.

**Step-by-Step Example**:
```javascript
let price = 10.5;

console.log(Math.round(price)); // 11
console.log(Math.floor(price)); // 10
console.log(Math.ceil(10.1));   // 11
```

---

### 2. Random Numbers & Formatting
**Theory**: Generating random numbers is crucial for games/logic, while formatting is needed for UI display.

**Key Methods**:
- `Math.random()`: Returns a random decimal between 0 (inclusive) and 1 (exclusive).
- `toFixed(n)`: Formats a number using fixed-point notation (returns a string).

**Step-by-Step Example**:
```javascript
// Step 1: Generate a random decimal
let randomDec = Math.random(); 

// Step 2: Multiply by 10 and floor it to get an integer from 0-9
let randomInt = Math.floor(randomDec * 10); 
console.log(randomInt);

// Formatting money
let cost = 5.56789;
console.log(cost.toFixed(2)); // "5.57" (Returns a string!)
```
