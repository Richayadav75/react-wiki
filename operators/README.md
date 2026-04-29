- Category: Fundamentals
- Track: Fundamentals
- Difficulty: Beginner
- Related: variables

### What are Operators?
Operators are symbols that perform operations on data. We categorize them by their purpose.

---

### 1. Arithmetic Operators
**Theory**: Used for mathematical calculations like addition and multiplication.

**Key Features**:
- **Basic Math**: `+`, `-`, `*`, `/`.
- **Remainder**: `%` (Modulus) gives the remainder of a division.
- **Increment**: `++` adds 1 to the value.

**Example**:
```javascript
let x = 10 + 5; // 15
let rem = 10 % 3; // 1 (Remainder of 10/3)
console.log(x, rem);
```

---

### 2. Comparison Operators
**Theory**: Used to compare two values, resulting in `true` or `false`.

**Working Flow**
```text
[ Value A ] --( Comparison Operator )--> [ Value B ]
      |                 |                    |
      v                 v                    v
  ( Check ) --------( Match? ) --------> [ Boolean Result ]
                                         (true / false)
```

**Key Features**:
- **Equality**: `===` (Strictly equal in value and type).
- **Greater/Less**: `>`, `<`, `>=`, `<=`.
- **Difference**: `!==` (Not equal).

**Example**:
```javascript
console.log(10 === "10"); // false (one is number, one is string)
console.log(5 > 2); // true
```

---

### 3. Logical Operators
**Theory**: Used to combine multiple boolean expressions into a single logic.

**Working Flow**
```text
( Cond 1 )  ( Cond 2 )
     |          |
     v          v
[ LOGICAL OPERATOR ]
          |
          v
   [ FINAL BOOLEAN ]
```

**Key Features**:
- **&& (AND)**: True only if BOTH sides are true.
- **|| (OR)**: True if AT LEAST ONE side is true.
- **! (NOT)**: Reverses the result (True becomes False).

**Example**:
```javascript
let isLogged = true;
let hasPermission = false;

if (isLogged && hasPermission) {
  console.log("Access Granted");
} else {
  console.log("Access Denied"); // This will run
}
```

---

[View Interview Questions](./interview.md)
