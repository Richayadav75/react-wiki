- Category: JavaScript
- Difficulty: Beginner
- Related: data-types

### What are String Methods?
Strings in JavaScript have built-in methods that allow you to search, slice, modify, and format text easily.

---

### 1. Extracting Parts of a String
**Theory**: You can extract a specific portion of a string using `slice()` or `substring()`. They return a **new string**.

**Working Flow**
```text
[ "J", "a", "v", "a", "S", "c", "r", "i", "p", "t" ]
   0    1    2    3    4    5    6    7    8    9
   ^-- slice(0, 4) --^ => "Java"
```

**Key Methods**:
- `slice(start, end)`: Extracts from `start` index up to (but not including) `end`. Accepts negative indexes.
- `substring(start, end)`: Similar to slice, but treats negative numbers as 0.

**Step-by-Step Example**:
```javascript
const text = "Hello World";

// Step 1: Start at index 0, stop BEFORE index 5
const greeting = text.slice(0, 5); 
console.log(greeting); // "Hello"

// Step 2: Provide only a start index to get the rest of the string
const world = text.slice(6); 
console.log(world); // "World"
```

---

### 2. Modifying Strings
**Theory**: Since strings are immutable, methods like `replace()`, `toUpperCase()`, and `trim()` return a modified **copy** of the string.

**Working Flow**
```text
"  hello  " --( trim )--> "hello" --( toUpperCase )--> "HELLO"
```

**Key Methods**:
- `toUpperCase()` / `toLowerCase()`: Changes case.
- `trim()`: Removes whitespace from both ends.
- `replace(search, new)`: Replaces the first match with new text.

**Step-by-Step Example**:
```javascript
const messyInput = "   learn REACT   ";

// Step 1: Clean the extra spaces
const clean = messyInput.trim(); // "learn REACT"

// Step 2: Make it all lowercase
const lower = clean.toLowerCase(); // "learn react"

// Step 3: Replace 'react' with 'javascript'
const final = lower.replace("react", "javascript"); // "learn javascript"
```

---

### 3. Searching Strings
**Theory**: Methods to check if a string contains specific text.

**Key Methods**:
- `includes()`: Returns true/false.
- `indexOf()`: Returns the position (index) of the first match, or -1.

**Step-by-Step Example**:
```javascript
const sentence = "The quick brown fox";

// Check if 'fox' is in the sentence
console.log(sentence.includes("fox")); // true

// Find exactly where 'quick' starts
console.log(sentence.indexOf("quick")); // 4
```
