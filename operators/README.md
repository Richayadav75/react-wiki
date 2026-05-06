- Category: Fundamentals
- Track: Fundamentals
- Difficulty: Beginner
- Related: variables

### What are Operators?
Operators are symbols that perform operations on data. We categorize them by their purpose.


### 1. Arithmetic Operators
**Theory**: These perform standard math. The % (modulo) and ** (exponent) operators are the ones new learners often miss.

**Key Features**:
- **Basic Math**: `+`, `-`, `*`, `/`.
- **Remainder**: `%` (Modulus) gives the remainder of a division.
- **Increment**: `++` adds 1 to the value.

**Example**:
```javascript
let a = 10, b = 3;

console.log(a + b);   // → 13  (addition)
console.log(a - b);   // → 7   (subtraction)
console.log(a * b);   // → 30  (multiplication)
console.log(a / b);   // → 3.33 (division)
console.log(a % b);   // → 1   (remainder/modulo)
console.log(a ** b);  // → 1000 (10 to the power 3)

let x = 5;
x++;           // x is now 6  (increment)
x--;           // x is now 5  (decrement)
console.log(x); // → 5

```

**Explanation**:

`a % b // → 1`
Modulo returns the remainder after division. 10 ÷ 3 = 3 remainder 1. Used to check even/odd, cycles, etc.

`a ** b // → 1000`
Exponentiation — 10³ = 10 × 10 × 10 = 1000.

`x++`
Increment shorthand — same as x = x + 1.

`x--`
Decrement shorthand — same as x = x - 1.


### 2. Comparison Operators
**Theory**: Used to compare two values, resulting in `true` or `false`.
Comparisons return true or false. The critical one to learn is === (strict equality) — it checks value AND type. Avoid == which does type coercion.



**Key Features**:
- **Equality**: `===` (Strictly equal in value and type).
- **Greater/Less**: `>`, `<`, `>=`, `<=`.
- **Difference**: `!==` (Not equal).

**Example**:
```javascript
console.log(5 === 5);     // → true  (strict equal)
console.log(5 === "5");   // → false (number ≠ string)
console.log(5 == "5");    // → true  (loose — coerces type)
console.log(5 !== 6);     // → true  (strict not equal)
console.log(10 > 3);      // → true
console.log(10 < 3);      // → false
console.log(5 >= 5);      // → true  (greater or equal)
console.log(4 <= 3);      // → false (less or equal)
```

**Explanation**:

`5 === 5 // true`
Strict equality: same value AND same type. Always prefer this.

`5 === "5" // false`
5 is a number, '5' is a string — different types so false.

`5 == "5" // true`
Loose equality: JS converts string to number first. Avoid ==.

`5 !== 6 // true`
Strict not-equal. Use !== instead of !=.


### 3. Logical Operators
**Theory**: Combine conditions. && (AND), || (OR), ! (NOT). They also return the actual values, not just true/false — useful for default values.

**Key Features**:
- **&& (AND)**: True only if BOTH sides are true.
- **|| (OR)**: True if AT LEAST ONE side is true.
- **! (NOT)**: Reverses the result (True becomes False).

**Example**:
```javascript
let age = 20, hasID = true;

// AND — both must be true
console.log(age >= 18 && hasID);  // → true

// OR — at least one must be true
console.log(age < 18 || hasID);   // → true

// NOT — flips boolean
console.log(!hasID);              // → false

// Practical: default value with ||
let username = "";
let display = username || "Guest"; // → "Guest"
console.log(display);

// Nullish coalescing (??) — only falls back for null/undefined
let count = 0;
let shown = count ?? 10;   // → 0  (0 is not null!)
console.log(shown);
```

**Explanation**:

`age >= 18 && hasID // true`
Both sides must be truthy. Think 'if this AND that'.

`age < 18 || hasID // true`
At least one side must be truthy. Think 'if this OR that'.

`!hasID // false`
NOT inverts the boolean. !true = false, !false = true.

`username || "Guest"`
If username is falsy (empty string), return 'Guest'. Common default value pattern.

`count ?? 10 // → 0`
?? only falls back on null or undefined. 0 and '' are kept as-is.


### 4. Assignment & Ternary Operators
**Theory**: Shorthand operators reduce repetition. The ternary operator is a one-line if/else.

**Example**:
```javascript
let x = 10;

x += 5;   // same as x = x + 5  → 15
x -= 3;   // same as x = x - 3  → 12
x *= 2;   // same as x = x * 2  → 24
x /= 4;   // same as x = x / 4  → 6

console.log(x);  // → 6

// Ternary: condition ? valueIfTrue : valueIfFalse
let score = 75;
let result = score >= 50 ? "Pass" : "Fail";
console.log(result);  // → "Pass"

```

**Explanation**:

`x += 5;`
Compound assignment. Adds 5 to x and saves back. Works for -=, *=, /= too.

`score >= 50 ? "Pass" : "Fail"`
Ternary operator — condition ? then : else. Returns a value directly.
---

[View Interview Questions](./interview.md)
