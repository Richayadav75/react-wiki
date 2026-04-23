- Category: JavaScript
- Difficulty: Beginner
- Related: Arrow Functions, Destructuring

# ES6+ Features

Modern JavaScript (ES6 and beyond) introduced many features that make coding more efficient and readable.

## Key Features
- **Arrow Functions**: Concise syntax for functions.
- **Destructuring**: Extract values from arrays or objects easily.
- **Spread/Rest Operator**: `...` for expanding or collecting elements.
- **Template Literals**: Backticks for string interpolation.
- **Let/Const**: Block-scoped variable declarations.

## Example
```javascript
const user = { name: 'Wiki', age: 25 };
const { name, age } = user; // Destructuring
console.log(`User ${name} is ${age} years old.`); // Template Literal
```

## Interview Questions
1. **What is the difference between let, const, and var?**
   `var` is function-scoped and hoisted. `let` and `const` are block-scoped and not initialized during hoisting (TDZ). `const` cannot be reassigned.
2. **What are arrow functions?**
   They are a shorter syntax for writing functions and they do not have their own `this`.
