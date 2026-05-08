- Category: JavaScript
- Track: JavaScript
- Difficulty: Beginner
- Related: Fundamentals

### What are Math & Number Methods?
JavaScript provides a built-in static `Math` object that contains properties and methods for mathematical constants and functions. Unlike many other objects, `Math` is not a constructor; you use its methods directly.

---

### 1. Random Number Generation Flow
**Working Flow: Getting an Integer between Min and Max**

```mermaid
graph LR
    A[Math.random] --> B["Multiply by (Max - Min + 1)"]
    B --> C[Add Min]
    C --> D[Math.floor]
    D --> E[Random Integer]
```

---

### 2. Core Math Methods

#### Rounding & Truncating
| Method | Description | Example |
| :--- | :--- | :--- |
| `round(x)` | Rounds to nearest integer | `Math.round(4.5) // 5` |
| `floor(x)` | Rounds DOWN | `Math.floor(4.9) // 4` |
| `ceil(x)` | Rounds UP | `Math.ceil(4.1) // 5` |
| `trunc(x)` | Removes decimals | `Math.trunc(-4.9) // -4` |

#### Comparison & Absolute
| Method | Description | Example |
| :--- | :--- | :--- |
| `abs(x)` | Returns absolute value | `Math.abs(-5) // 5` |
| `min(...n)` | Returns smallest arg | `Math.min(1, 5, -2) // -2` |
| `max(...n)` | Returns largest arg | `Math.max(1, 5, -2) // 5` |

---

### 3. Comprehensive Examples

#### Generating a Random Range
**Theory**: `Math.random()` returns a number between 0 and 1. To get an integer in a specific range, use the following formula.
```javascript
function getRandomInt(min, max) {
  return Math.floor(Math.random() * (max - min + 1)) + min;
}

console.log(getRandomInt(1, 10)); // Random number from 1 to 10
```

#### Finding Max in an Array
```javascript
const scores = [82, 95, 71, 88];
const topScore = Math.max(...scores); // Use spread operator
console.log(topScore); // 95
```
**Output**: `95`

---

### 4. Comparison: floor() vs trunc()
While they seem similar, they behave differently with negative numbers:
- `Math.floor(-4.1)` → `-5` (Rounds down to the next lower integer)
- `Math.trunc(-4.1)` → `-4` (Simply removes the fractional part)

---

[View Interview Questions](./interview.md)
