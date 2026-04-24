# "this" Keyword Interview Questions

1. **What does "this" refer to in an arrow function?**
   - It refers to the `this` value of the enclosing lexical scope (it doesn't have its own `this`).

2. **How do call, apply, and bind differ?**
   - `call` and `apply` execute the function immediately (call takes comma-separated args, apply takes an array). `bind` returns a new function with a fixed `this` for later execution.

3. **What is the default binding of "this" in strict mode?**
   - It is `undefined` instead of the `window` (global) object.
