- Category: Fundamentals
- Track: Fundamentals
- Difficulty: Beginner
- Related: operators

### What Implicit type coercion?
**Theory**: JavaScript automatically converts types in certain situations. This is called implicit coercion and is a major source of bugs if you don't understand it.


**Example**:
```javascript
// String + Number = String (concatenation wins)
console.log("5" + 3);      // → "53"  (not 8!)
console.log("5" + true);   // → "5true"

// Other operators convert string to number
console.log("5" - 2);      // → 3
console.log("5" * 2);      // → 10
console.log("abc" * 2);    // → NaN (Not a Number)

// Truthy and falsy values
if ("hello") console.log("truthy!");  // → truthy!
if (0)       console.log("never");   // 0 is falsy

```

**Explanation**:

`"5" + 3  // → "53"`
JS converts 3 to "3" before adding. String concatenation wins over arithmetic.

`"5" - 2  // → 3`
No string-to-number conversion rule here, so JS treats both as numbers.

`"abc" * 2 // → NaN`
"abc" cannot be converted to a number, so multiplication fails → NaN.

`"hello"   // truthy`
Any non-empty string is truthy. It coerces to true in conditions.

`if (0) ...        // falsy`
Zero is one of the few values that coerces to false.
**Output**:
```
53
5true
3
10
NaN
truthy!
never
```

### 2. what is explicit type coercion?
**Theory**: Always convert explicitly when mixing types. Number(), String(), Boolean() are the safe tools. typeof tells you what type something is.

**Example**:
```javascript
// Convert to Number
console.log(Number("42"));    // → 42
console.log(Number("3.14")); // → 3.14
console.log(Number(""));     // → 0
console.log(Number("hi"));   // → NaN
console.log(parseInt("10px")); // → 10 (stops at non-digit)

// Convert to String
console.log(String(42));     // → "42"
console.log(String(true));   // → "true"
console.log((42).toString()); // → "42"

// Convert to Boolean
console.log(Boolean(0));     // → false
console.log(Boolean("hi"));  // → true
console.log(Boolean(null));  // → false

// typeof
console.log(typeof 42);      // → "number"
console.log(typeof "hi");    // → "string"
console.log(typeof true);    // → "boolean"
console.log(typeof undefined); // → "undefined"

```

**Explanation**:

`Number("42") // → 42`
Converts string to number. Returns NaN if invalid.

`parseInt("10px") // → 10`
Parses integer from start of string. Stops at non-digits.

`String(42) // → "42"`
Converts anything to string.

`(42).toString() // → "42"`
Alternative way for numbers.

`Boolean(0) // → false`
Boolean(""), null, undefined, 0, NaN are falsy. Everything else is truthy.

`typeof 42 // → "number"`
Returns the type as a string. Note: typeof null is "object" (a bug).
**Output**:
```
42
3.14
0
NaN
10
"42"
"true"
"42"
false
true
false
"number"
"string"
"boolean"
"undefined"
```

[View Interview Questions](./interview.md)
