# Math Methods — Interview Questions

---

**1. What is the difference between `Math.round()`, `Math.ceil()`, `Math.floor()`, and `Math.trunc()`?**

| Method | Behavior | `4.1` | `4.9` | `-4.1` | `-4.9` |
|--------|----------|-------|-------|--------|--------|
| `round` | Nearest integer (.5 rounds up) | 4 | 5 | -4 | -5 |
| `ceil` | Always rounds UP | 5 | 5 | -4 | -4 |
| `floor` | Always rounds DOWN | 4 | 4 | -5 | -5 |
| `trunc` | Just removes decimal | 4 | 4 | -4 | -4 |

The key difference: `floor` and `trunc` differ for **negative numbers**.
- `Math.floor(-4.1)` → `-5` (floor goes further negative)
- `Math.trunc(-4.1)` → `-4` (trunc just drops decimal, same direction as positive)

---

**2. How do you generate a random integer in a specific range?**

```javascript
function randInt(min, max) {
  return Math.floor(Math.random() * (max - min + 1)) + min;
}

randInt(1, 6);    // → 1,2,3,4,5, or 6  (dice roll)
randInt(10, 99);  // → random two-digit number
randInt(0, 1);    // → 0 or 1 (coin flip)
```

Step by step for range 1–6:
1. `Math.random()` → e.g. `0.73`
2. `× (6 - 1 + 1) = × 6` → `4.38`
3. `Math.floor(4.38)` → `4`
4. `+ 1` → `5`

---

**3. How do you find the max/min value in an array?**

`Math.max()` and `Math.min()` accept individual arguments, so you must spread the array:

```javascript
const scores = [82, 95, 71, 88, 60];

Math.max(...scores); // → 95
Math.min(...scores); // → 60

// For large arrays, reduce can be more efficient
scores.reduce((max, n) => n > max ? n : max, -Infinity); // → 95
```

**Important**: `Math.max([82, 95])` without spread returns `NaN` — it receives a single array argument, not numbers.

---

**4. What is the difference between `Math.floor()` and `Math.trunc()` for negative numbers?**

```javascript
Math.floor(-4.1);  // → -5  (rounds DOWN — more negative)
Math.trunc(-4.1);  // → -4  (just removes decimal)

Math.floor(-4.9);  // → -5
Math.trunc(-4.9);  // → -4

// For positive numbers, they behave identically
Math.floor(4.9);   // → 4
Math.trunc(4.9);   // → 4
```

Use `trunc` when you want to truncate without considering direction. Use `floor` when you always want the mathematically lower integer.

---

**5. How do you round a number to a specific number of decimal places?**

```javascript
// Multiply, round, divide
const price = 19.9955;
const rounded = Math.round(price * 100) / 100;
console.log(rounded); // → 20

// toFixed — returns a STRING
(19.9955).toFixed(2);  // → "20.00" (string!)

// For display
`$${(price).toFixed(2)}`; // → "$19.99"

// For math — convert back to number
Number((price).toFixed(2)); // → 19.99  (number)
+(price).toFixed(2);         // → 19.99  (number)
```

---

**6. What is the clamp pattern and how do you implement it?**

Clamping ensures a value stays within a min/max boundary — very common for progress bars, volume controls, and input validation.

```javascript
function clamp(value, min, max) {
  return Math.min(Math.max(value, min), max);
}

clamp(150, 0, 100); // → 100  (too high → capped at max)
clamp(-5,  0, 100); // → 0    (too low → raised to min)
clamp(75,  0, 100); // → 75   (in range → unchanged)

// Progress bar usage
function progressPercent(done, total) {
  return clamp(Math.round((done / total) * 100), 0, 100);
}
progressPercent(12, 10); // → 100 (won't exceed 100%)
```

---

**7. How do you pick a random item from an array?**

```javascript
function randomItem(arr) {
  return arr[Math.floor(Math.random() * arr.length)];
}

const fruits = ["apple", "banana", "mango", "grape"];
randomItem(fruits); // → one of the four fruits (random)
```

`Math.random() * arr.length` gives a float from `0` to `3.999...`. `Math.floor` converts it to `0`, `1`, `2`, or `3` — valid indices.

---

**8. How does `Math.abs()` work and what are its common use cases?**

`Math.abs(x)` returns the non-negative value of `x`:

```javascript
Math.abs(-7);   // → 7
Math.abs(7);    // → 7
Math.abs(-3.5); // → 3.5

// Distance between two points
const diff = Math.abs(a - b); // works regardless of which is larger

// Percentage change (always positive)
function percentChange(from, to) {
  return (Math.abs(to - from) / from * 100).toFixed(1) + "%";
}
percentChange(100, 75); // → "25.0%"

// Float equality check (floating-point safety)
Math.abs(0.1 + 0.2 - 0.3) < 0.0001; // → true
```

---

**9. Is `Math` a constructor? How is it different from other built-in objects?**

No — `Math` is a plain static object (namespace), not a constructor. You cannot call `new Math()`.

```javascript
typeof Math;      // → "object"
new Math();       // → TypeError: Math is not a constructor

// Compare with Array/Date (constructors):
new Array(3);  // → [empty × 3]
new Date();    // → current date
```

All `Math` methods and constants (`Math.PI`, `Math.E`) are accessed directly on the `Math` object. There are no instances.

---

**10. How do you generate a random 6-digit OTP?**

```javascript
function generateOTP(digits = 6) {
  const min = 10 ** (digits - 1);      // 100000 for 6 digits
  const max = 10 ** digits - 1;        // 999999 for 6 digits
  return String(Math.floor(Math.random() * (max - min + 1)) + min);
}

console.log(generateOTP());   // → "483920"
console.log(generateOTP(4));  // → "7341"

// Alternative using padStart (can start with 0)
function generateOTPv2(digits = 6) {
  return String(Math.floor(Math.random() * 10 ** digits))
    .padStart(digits, "0");
}
console.log(generateOTPv2(4)); // → "0394" (can have leading zeros)
```
