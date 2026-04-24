- Category: Fundamentals
- Difficulty: Beginner
- Related: operators

### What are Conditionals?
Conditionals allow the program to make decisions by executing different code blocks based on conditions.

---

### 1. If / Else
**Theory**: The fundamental decision-making structure.

**Working Flow**
```text
[ Start ] --> ( Check Condition ) --> [ Path A (True) ]
                          | 
                          L-> [ Path B (False) ]
```

**Key Features**:
- **if**: Runs if condition is true.
- **else**: Runs if condition is false.

**Example**:
```javascript
if (score > 50) {
  console.log("Pass");
} else {
  console.log("Fail");
}
```

---

### 2. Else If (Multi-branch)
**Theory**: Used to test several conditions in sequence.

**Key Features**:
- **Chain**: Checks one by one until it finds a true condition.
- **Fallback**: Ends with an optional `else`.

**Example**:
```javascript
if (marks > 90) console.log("A+");
else if (marks > 75) console.log("A");
else console.log("B");
```
