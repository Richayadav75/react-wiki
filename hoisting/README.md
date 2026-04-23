- Category: Core Concepts of JavaScript
- Difficulty: Beginner
- Related: Closures, Scope

# Hoisting

Hoisting is JavaScript's default behavior of moving declarations to the top of the current scope.

## What is Hoisted?
1. **Variable Declarations**: `var` is hoisted and initialized with `undefined`. `let` and `const` are hoisted but not initialized (Temporal Dead Zone).
2. **Function Declarations**: Fully hoisted, meaning you can call them before they are defined.
3. **Class Declarations**: Not hoisted.

## Example
```javascript
console.log(x); // undefined
var x = 5;

sayHello(); // "Hello!"
function sayHello() {
  console.log("Hello!");
}
```
