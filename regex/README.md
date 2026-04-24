- Category: JavaScript
- Difficulty: Intermediate
- Related: string-methods

### What are Regular Expressions?
Regular Expressions (Regex) are patterns used to match character combinations in strings. They are extremely powerful for form validation (like checking if an email is valid) or searching text.

---

### 1. Defining a Pattern
**Theory**: A regex pattern is written between slashes `/pattern/` followed by optional flags like `i` (case-insensitive) or `g` (global search).

**Working Flow**
```text
[ Input String ] ---> ( test(/pattern/) ) ---> [ true / false ]
```

**Key Methods**:
- `test()`: Returns true if the string contains a match.
- `match()`: Returns an array containing all of the matches.

**Step-by-Step Example**:
```javascript
// Step 1: Define a pattern. 
// ^ starts with 'A', .* means any characters, i means case-insensitive
const pattern = /^A.*/i; 

// Step 2: Test a string against the pattern
const result1 = pattern.test("Apple");  // true
const result2 = pattern.test("Banana"); // false

console.log(result1, result2);
```

---

### 2. Common Validation (Email)
**Theory**: Regex is the standard way to validate user input before sending it to a server.

**Step-by-Step Example**:
```javascript
// A standard (simplified) email validation regex
const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

function validateEmail(email) {
  // Step 1: Test the input email against the regex
  if (emailRegex.test(email)) {
    console.log("Valid email format!");
  } else {
    console.log("Invalid email!");
  }
}

validateEmail("test@example.com"); // "Valid email format!"
validateEmail("test_example.com"); // "Invalid email!"
```

---

[View Interview Questions](./interview.md)
