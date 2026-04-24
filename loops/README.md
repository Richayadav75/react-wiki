- Category: Fundamentals
- Difficulty: Beginner
- Related: conditionals

### What are Loops?
Loops repeat a block of code multiple times. This is essential for handling lists or repetitive tasks.

---

### 1. For Loop
**Theory**: Best when you know exactly how many times you want to repeat.

**Working Flow**
```text
[ Init ] --> ( Condition? ) --> [ Run Code ] --> [ Increment ] --|
                   ^                                         |
                   |-----------------------------------------|
```

**Key Features**:
- **Initializer**: Where it starts.
- **Condition**: When it stops.
- **Step**: How it changes each time.

**Example**:
```javascript
for (let i = 1; i <= 5; i++) {
  console.log("Number: " + i);
}
```

---

### 2. While Loop
**Theory**: Runs as long as a condition is true. Best for unknown durations.

**Key Features**:
- **Condition-first**: Checks before running code.
- **Risk**: Can become an infinite loop if condition never fails.

**Example**:
```javascript
let energy = 3;
while (energy > 0) {
  console.log("Working...");
  energy--;
}
```

---

[View Interview Questions](./interview.md)
