# Hoisting Interview Questions

1. **What is variable hoisting?**
   - JavaScript's behavior of moving declarations to the top of the scope during the compile phase.

2. **Difference between var, let, and const hoisting?**
   - `var` is hoisted and initialized with `undefined`. `let` and `const` are hoisted but stay in the "Temporal Dead Zone" (uninitialized) until the code execution reaches their declaration.

3. **Are functions hoisted?**
   - Function declarations are fully hoisted. Function expressions (e.g., `const x = function() {}`) are not.
