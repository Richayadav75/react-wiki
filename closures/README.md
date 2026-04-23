- Category: JavaScript
- Difficulty: Intermediate
- Related: Scope, Hoisting

# Closures

A closure is the combination of a function bundled together (enclosed) with references to its surrounding state (the lexical environment). In other words, a closure gives you access to an outer function's scope from an inner function.

## How it works
In JavaScript, closures are created every time a function is created, at function creation time.

## Example
```javascript
function makeAdder(x) {
  return function (y) {
    return x + y;
  };
}

const add5 = makeAdder(5);
console.log(add5(2)); // 7
```

## Practice
```javascript
function createCounter() {
  let count = 0;
  return {
    increment: () => ++count,
    decrement: () => --count,
    getCount: () => count
  };
}

const counter = createCounter();
console.log(counter.increment()); // 1
console.log(counter.increment()); // 2
```

## Interview Questions
1. **What is a closure?**
   A closure is a function that remembers its outer variables and can access them.
2. **What are the common use cases for closures?**
   Data privacy, function factories, and maintaining state in asynchronous callbacks.
