- Category: Fundamentals
- Track: Fundamentals
- Difficulty: Beginner
- Related: operators

### What are Conditionals?
Conditionals allow the program to make decisions by executing different code blocks based on conditions.

---

### 1. If / Else
**Theory**: The most fundamental way to make decisions in code. JS evaluates the condition in () and runs the matching block.


**Key Features**:
- **if**: Runs if condition is true.
- **else**: Runs if condition is false.

**Example**:
```javascript
let temperature = 35;

if (temperature > 40) {
  console.log("Extreme heat!");
} else if (temperature > 30) {
  console.log("Hot day");     // ← this runs
} else if (temperature > 20) {
  console.log("Warm day");
} else {
  console.log("Cool or cold");
}

// One-liner (no braces) — only for single statements
let x = 10;
if (x > 5) console.log("big");  // → big

```

**Explanation**:

`if (temperature > 40) {`
Evaluates the condition. If true, runs this block and skips all others.

`} else if (temperature > 30) {`
Only checked if the first if was false. 35 > 30, so this runs.

`} else {`
Fallback — runs only if every condition above was false.

`if (x > 5) console.log("big");`
Single-line form. Fine for simple cases, use braces for multi-line.

---

### 2. Switch
**Theory**: switch is cleaner than many else-if chains when matching one variable against exact values. The break stops fall-through.

**Key Features**:
- **Cases**: Each condition is a `case` label.
- **Break**: CRITICAL! If omitted, code falls through to the next case (danger zone!).
- **Default**: Runs if no case matches (like `else`).

**Example**:
```javascript
let day = "Monday";

switch (day) {
  case "Saturday":
  case "Sunday":
    console.log("Weekend!");
    break;
  case "Monday":
    console.log("Start of week"); // ← runs
    break;
  case "Friday":
    console.log("Almost there!");
    break;
  default:
    console.log("Weekday");
}

```

**Explanation**:

`switch (day) {`
Evaluates day once, then jumps to the matching case.

`case "Saturday": case "Sunday":`
Two cases with no break between them — both lead to the same block (fall-through on purpose).

`break;`
Exits the switch. Without break, execution falls through to the next case.

`default:`
Runs when no case matches. Like else in if/else.
### 3. for loop 
**Theory**: The classic loop. Three parts in the (): initializer, condition checked before each pass, update run after each pass.


**Example**:
```javascript
// Basic for loop — count 1 to 5
for (let i = 1; i <= 5; i++) {
  console.log(i);     // → 1 2 3 4 5
}

// Loop over an array
let fruits = ["apple", "banana", "mango"];
for (let i = 0; i < fruits.length; i++) {
  console.log(fruits[i]);
}

// for...of — cleaner for arrays (modern)
for (let fruit of fruits) {
  console.log(fruit);
}

```

**Explanation**:

`let i = 1; i <= 5; i++`
Start at 1. Keep going while i <= 5. Add 1 after each iteration.

`i < fruits.length`
fruits.length is 3. Loop runs for i=0, i=1, i=2 then stops.

`fruits[i]`
Access array element by index. fruits[0] = 'apple', fruits[1] = 'banana', etc.

`for (let fruit of fruits)`
for...of gives you the value directly — no index needed. Use for arrays.

### 4. while & do-while loops
**Theory**: while checks the condition before each iteration. do-while always runs at least once — then checks.
**Example**:
```javascript
// while — runs while condition is true
let n = 1;
while (n <= 3) {
  console.log(n);   // → 1 2 3
  n++;
}

// do-while — runs at least once
let attempt = 1;
do {
  console.log("Attempt: " + attempt);
  attempt++;
} while (attempt <= 3);

// break & continue
for (let i = 0; i < 5; i++) {
  if (i === 2) continue;   // skip 2
  if (i === 4) break;      // stop at 4
  console.log(i);          // → 0 1 3
}

```

**Explanation**:

`while (n <= 3) {`
Checks n <= 3 before entering. Once n becomes 4, exits.

`do { ... } while (attempt <= 3);`
Body runs first. Then checks. Useful for menus, retries.

`if (i === 2) continue;`
Skip the rest of this iteration, go to next i.

`if (i === 4) break;`
Exit the entire loop immediately.


---

[View Interview Questions](./interview.md)
