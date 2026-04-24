- Category: JavaScript
- Difficulty: Beginner
- Related: promises

### What is Error Handling?
Errors happen! Network requests fail, variables might be undefined, or users enter bad data. Error handling prevents your entire script from crashing by gracefully catching and dealing with these issues.

---

### 1. Try...Catch
**Theory**: You wrap risky code in a `try` block. If it fails, execution immediately jumps to the `catch` block instead of killing the program.

**Working Flow**
```text
[ try { riskyCode() } ] --( SUCCESS )--> [ Continue Normal Flow ]
          |
     ( ERROR! )
          |
          v
[ catch (err) { handleIt() } ] ------> [ Continue Normal Flow ]
```

**Key Features**:
- **try**: Contains code that might throw an error.
- **catch**: Contains code that executes ONLY if an error occurs.
- **finally**: (Optional) Code that runs regardless of success or failure.

**Step-by-Step Example**:
```javascript
// Step 1: Wrap risky code in try
try {
  let user = JSON.parse("{ bad json }"); // This will fail!
  console.log("This line will never run.");
  
} catch (error) {
  // Step 2: Handle the error gracefully
  console.log("Oops! We caught an error: " + error.name);
  
} finally {
  // Step 3: Clean up (runs no matter what)
  console.log("Execution finished.");
}
```

---

### 2. Custom Errors (Throw)
**Theory**: You don't have to wait for JavaScript to throw an error. You can manually `throw` your own errors if data is invalid.

**Working Flow**
```text
[ Check Data ] --( Invalid )--> [ throw new Error("Bad Data") ]
```

**Step-by-Step Example**:
```javascript
function calculateAge(yearBorn) {
  // Step 1: Check for invalid input
  if (yearBorn > 2024) {
    // Step 2: Manually throw an error
    throw new Error("Year cannot be in the future!");
  }
  return 2024 - yearBorn;
}

try {
  calculateAge(2050); // Triggers our custom error
} catch (err) {
  console.log("Caught custom error: " + err.message);
}
```
