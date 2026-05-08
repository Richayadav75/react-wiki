# ES6+ Features Interview Questions

1. **What is Destructuring in ES6?**
   - It is a syntax that allows you to unpack values from arrays or properties from objects into distinct variables in a very concise way.

2. **Explain the Spread and Rest operators.**
   - **Spread (`...`)**: Used to "expand" an iterable (like an array) into individual elements. Useful for cloning or merging arrays/objects.
   - **Rest (`...`)**: Used in function parameters to "gather" all remaining arguments into a single array.

3. **What are Template Literals?**
   - They are string literals allowing embedded expressions. You use backticks (``) and `${expression}` for interpolation. They also support multi-line strings without `\n`.

4. **What are the advantages of Arrow Functions?**
   - Shorter syntax.
   - Implicit return (for one-liners).
   - Lexical `this`: They don't have their own `this`, making them perfect for callbacks where you want to preserve the parent context.

5. **What is the difference between `let`, `const`, and `var`?**
   - `var`: Function-scoped, hoisted, can be re-declared.
   - `let`: Block-scoped, not re-declarable, helps avoid TDZ issues.
   - `const`: Block-scoped, must be initialized immediately, cannot be reassigned.

6. **What are Default Parameters?**
   - They allow you to initialize function parameters with default values if no value or `undefined` is passed to the function.

7. **How do you merge two objects in ES6?**
   - Using the spread operator: `const merged = { ...obj1, ...obj2 };`
