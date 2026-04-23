- Category: JavaScript
- Difficulty: Beginner
- Related: ES6 Features

# Array Methods

JavaScript provides powerful methods to manipulate arrays without using traditional for-loops.

## Common Methods
- **map()**: Creates a new array with the results of calling a function on every element.
- **filter()**: Creates a new array with all elements that pass a test.
- **reduce()**: Executes a reducer function on each element, resulting in a single output value.
- **forEach()**: Executes a function once for each array element.

## Example
```javascript
const numbers = [1, 2, 3, 4];
const doubled = numbers.map(n => n * 2); // [2, 4, 6, 8]
const evens = numbers.filter(n => n % 2 === 0); // [2, 4]
```

## Interview Questions
1. **What is the difference between map() and forEach()?**
   `map()` returns a new array, while `forEach()` returns `undefined` and is used for side effects.
2. **How does reduce() work?**
   It takes an accumulator and a current value, applying a function to combine them into a single result.
