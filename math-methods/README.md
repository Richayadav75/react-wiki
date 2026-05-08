- Category: JavaScript
- Difficulty: Beginner
- Related: variables, data-types, functions

### JavaScript Math Methods — Numbers and Calculations
JavaScript's built-in `Math` object is a toolbox of mathematical functions and constants. Unlike most objects, `Math` is not a constructor — you never do `new Math()`. You call methods directly: `Math.round()`, `Math.max()`, etc.

**Analogy**
`Math` is like a scientific calculator that's always on your desk. You don't build a new calculator each time — you just press the button you need. Every function is immediately available.

---

### 1. Rounding — round / ceil / floor / trunc
**Theory**: Four methods for converting decimals to integers, each with different rounding behavior. Choosing the right one matters for financial calculations, pagination, and UI display.

**Working Flow**
![flow-chart](flow-chart.png)

**Example**
```javascript
// round — nearest integer
Math.round(4.4);   // → 4
Math.round(4.5);   // → 5
Math.round(4.9);   // → 5
Math.round(-4.5);  // → -4  (rounds toward +Infinity)

// ceil — always up (ceiling)
Math.ceil(4.0);    // → 4
Math.ceil(4.1);    // → 5
Math.ceil(-4.9);   // → -4  (up for negatives = toward zero)

// floor — always down
Math.floor(4.9);   // → 4
Math.floor(-4.1);  // → -5  (down for negatives = away from zero)

// trunc — simply removes the decimal part
Math.trunc(4.9);   // → 4
Math.trunc(-4.9);  // → -4  (different from floor!)

// Practical: round to 2 decimal places
const price = 19.9955;
const rounded = Math.round(price * 100) / 100;
console.log(rounded); // → 20
```

**Output**
```
Math.round(4.4)   → 4
Math.round(4.5)   → 5
Math.ceil(4.1)    → 5
Math.ceil(-4.9)   → -4
Math.floor(4.9)   → 4
Math.floor(-4.1)  → -5
Math.trunc(4.9)   → 4
Math.trunc(-4.9)  → -4
rounded price     → 20
```

**Key difference** — `floor` vs `trunc` only differs for **negative numbers**: `floor(-4.1)` → `-5`, `trunc(-4.1)` → `-4`.

---

### 2. Math.max / Math.min — Finding Extremes
**Theory**: `Math.max()` and `Math.min()` accept any number of arguments and return the largest or smallest. Use the spread operator (`...`) to apply them to arrays.

**Working Flow**
![flow-chart-2](flow-chart-2.png)

**Example**
```javascript
// Basic usage
Math.max(10, 30, 20);    // → 30
Math.min(10, 30, 20);    // → 10

// With negative numbers
Math.max(-5, -1, -10);   // → -1
Math.min(-5, -1, -10);   // → -10

// From array (must use spread)
const temperatures = [22, 35, 18, 29, 41, 15];
const hottest  = Math.max(...temperatures); // → 41
const coldest  = Math.min(...temperatures); // → 15

// Find max from array of objects
const products = [
  { name: "Phone",  price: 800 },
  { name: "Tablet", price: 400 },
  { name: "Laptop", price: 1200 },
];
const maxPrice = Math.max(...products.map(p => p.price));
const minPrice = Math.min(...products.map(p => p.price));
console.log(maxPrice); // → 1200
console.log(minPrice); // → 400

// Clamp a value between min and max
function clamp(value, min, max) {
  return Math.min(Math.max(value, min), max);
}
console.log(clamp(150, 0, 100)); // → 100  (too high, clamped to max)
console.log(clamp(-5,  0, 100)); // → 0    (too low, clamped to min)
console.log(clamp(50,  0, 100)); // → 50   (in range, unchanged)
```

**Output**
```
Math.max(10,30,20)          → 30
Math.min(-5,-1,-10)         → -10
hottest                     → 41
coldest                     → 15
maxPrice (products)         → 1200
minPrice (products)         → 400
clamp(150,0,100)            → 100
clamp(-5,0,100)             → 0
clamp(50,0,100)             → 50
```

---

### 3. Math.abs — Absolute Value
**Theory**: `Math.abs(x)` returns the magnitude of a number without sign. Useful for calculating distances, differences, and ensuring non-negative values.

**Working Flow**
![flow-chart-3](flow-chart-3.png)

**Example**
```javascript
Math.abs(5);    // → 5
Math.abs(-5);   // → 5
Math.abs(0);    // → 0
Math.abs(-3.7); // → 3.7

// Distance between two points on a number line
function distance(a, b) {
  return Math.abs(a - b);
}
distance(10, 3);  // → 7  (same as distance(3, 10))
distance(3, 10);  // → 7

// Percentage change
function percentChange(oldVal, newVal) {
  return Math.abs((newVal - oldVal) / oldVal * 100).toFixed(1) + "%";
}
console.log(percentChange(200, 150)); // → "25.0%"
console.log(percentChange(150, 200)); // → "33.3%"

// Check if value is close enough (within tolerance)
function isApproxEqual(a, b, tolerance = 0.0001) {
  return Math.abs(a - b) <= tolerance;
}
console.log(isApproxEqual(0.1 + 0.2, 0.3)); // → true
```

**Output**
```
Math.abs(-5)             → 5
distance(10,3)           → 7
distance(3,10)           → 7
percentChange(200,150)   → "25.0%"
percentChange(150,200)   → "33.3%"
isApproxEqual(0.1+0.2,0.3)→ true
```

---

### 4. Math.pow / Math.sqrt / Math.cbrt — Powers and Roots
**Theory**: `Math.pow(base, exp)` raises base to a power. The `**` operator does the same (ES2016). `Math.sqrt(x)` returns the square root. `Math.cbrt(x)` returns the cube root.

**Working Flow**
![flow-chart-4](flow-chart-4.png)

**Example**
```javascript
// Powers
Math.pow(2, 8);   // → 256
Math.pow(10, 3);  // → 1000
2 ** 8;           // → 256  (ES2016 shorthand)

// Square root
Math.sqrt(25);    // → 5
Math.sqrt(2);     // → 1.4142135623730951
Math.sqrt(-1);    // → NaN  (not real)

// Cube root
Math.cbrt(27);    // → 3
Math.cbrt(125);   // → 5
Math.cbrt(-8);    // → -2  (unlike sqrt, negative works)

// Hypotenuse (Pythagorean theorem)
function hypotenuse(a, b) {
  return Math.sqrt(Math.pow(a, 2) + Math.pow(b, 2));
  // or: Math.sqrt(a**2 + b**2)
}
console.log(hypotenuse(3, 4)); // → 5
console.log(hypotenuse(5, 12)); // → 13

// Compound interest: P * (1 + r)^t
function compoundInterest(principal, rate, years) {
  return (principal * Math.pow(1 + rate, years)).toFixed(2);
}
console.log(compoundInterest(10000, 0.08, 5));
// → "14693.28" (₹10,000 at 8% for 5 years)
```

**Output**
```
Math.pow(2,8)            → 256
Math.sqrt(25)            → 5
Math.sqrt(2)             → 1.4142135623730951
Math.cbrt(27)            → 3
hypotenuse(3,4)          → 5
hypotenuse(5,12)         → 13
compoundInterest(10000,0.08,5) → "14693.28"
```

---

### 5. Math.random — Generating Random Numbers
**Theory**: `Math.random()` returns a floating-point number between `0` (inclusive) and `1` (exclusive). By itself it's rarely useful — you need a formula to get random integers in a specific range.

**Working Flow**
![flow-chart-5](flow-chart-5.png)

**Example**
```javascript
// Raw random
Math.random();              // → 0.4721... (different each time)

// Integer 0 to 9
Math.floor(Math.random() * 10);   // → 0,1,2,...,9

// Integer 1 to 10
Math.floor(Math.random() * 10) + 1; // → 1,2,...,10

// Integer in range [min, max] inclusive
function randInt(min, max) {
  return Math.floor(Math.random() * (max - min + 1)) + min;
}
randInt(1, 6);   // → simulates a dice roll
randInt(10, 99); // → random two-digit number

// Random float in range
function randFloat(min, max) {
  return Math.random() * (max - min) + min;
}
randFloat(1.5, 3.5); // → e.g. 2.847...

// Pick a random item from an array
function randomItem(arr) {
  return arr[Math.floor(Math.random() * arr.length)];
}
const colors = ["red", "green", "blue", "yellow"];
randomItem(colors); // → one of the four colors

// Shuffle an array (Fisher-Yates)
function shuffle(arr) {
  const a = [...arr]; // don't mutate original
  for (let i = a.length - 1; i > 0; i--) {
    const j = randInt(0, i);
    [a[i], a[j]] = [a[j], a[i]]; // swap
  }
  return a;
}
console.log(shuffle([1, 2, 3, 4, 5]));
// → [3,1,5,2,4]  (random order)
```

**Output**
```
Math.random()              → 0.472... (varies)
floor(random()*10)         → 0-9
randInt(1,6)               → 1,2,3,4,5, or 6
randomItem(colors)         → "red"|"green"|"blue"|"yellow"
shuffle([1,2,3,4,5])       → [3,1,5,2,4] (random)
```

---

### 6. Math.PI and Other Constants
**Theory**: `Math` provides several important mathematical constants as properties.

**Working Flow**
![flow-chart-6](flow-chart-6.png)

**Example**
```javascript
// Circle calculations
function circleArea(radius) {
  return Math.PI * radius ** 2;
}
function circumference(radius) {
  return 2 * Math.PI * radius;
}

console.log(circleArea(5).toFixed(2));     // → "78.54"
console.log(circumference(5).toFixed(2)); // → "31.42"

// Degrees to radians and back
const toRadians = deg => (deg * Math.PI) / 180;
const toDegrees = rad => (rad * 180) / Math.PI;

console.log(toRadians(180)); // → 3.14159...  (= π)
console.log(toDegrees(Math.PI)); // → 180

// Logarithm
Math.log(Math.E);  // → 1  (natural log of e)
Math.log2(8);      // → 3  (log base 2 of 8)
Math.log10(1000);  // → 3  (log base 10 of 1000)
```

**Output**
```
circleArea(5)       → "78.54"
circumference(5)    → "31.42"
toRadians(180)      → 3.14159...
toDegrees(Math.PI)  → 180
Math.log2(8)        → 3
Math.log10(1000)    → 3
```

---

### Real-World Practical Patterns
```javascript
// 1. Percentage calculation (e.g., discount display)
function percentOf(part, total) {
  if (total === 0) return 0;
  return Math.round((part / total) * 100);
}
console.log(percentOf(73, 100));  // → 73
console.log(percentOf(1, 3));     // → 33

// 2. Rating stars (round to nearest 0.5)
function roundHalf(value) {
  return Math.round(value * 2) / 2;
}
console.log(roundHalf(3.3));  // → 3.5
console.log(roundHalf(4.1));  // → 4.0
console.log(roundHalf(4.7));  // → 5.0

// 3. Clamp progress bar (0% to 100%)
function progressPercent(completed, total) {
  const raw = (completed / total) * 100;
  return Math.min(100, Math.max(0, Math.round(raw)));
}
console.log(progressPercent(7, 10));   // → 70
console.log(progressPercent(12, 10));  // → 100  (clamped)
console.log(progressPercent(-1, 10));  // → 0    (clamped)

// 4. Generate a random OTP
function generateOTP(digits = 6) {
  const min = Math.pow(10, digits - 1);
  const max = Math.pow(10, digits) - 1;
  return String(Math.floor(Math.random() * (max - min + 1)) + min);
}
console.log(generateOTP());    // → "483920" (6-digit random)
console.log(generateOTP(4));   // → "7341"   (4-digit random)
```

**Output**
```
percentOf(73,100)      → 73
percentOf(1,3)         → 33
roundHalf(3.3)         → 3.5
progressPercent(7,10)  → 70
progressPercent(12,10) → 100
generateOTP()          → "483920" (random)
generateOTP(4)         → "7341"   (random)
```

---

[View Interview Questions](./interview.md)
