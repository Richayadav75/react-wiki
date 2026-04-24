# Variables Interview Questions

1. **What is the difference between var, let, and const?**
   - `var` is function-scoped, can be re-declared, and is hoisted with `undefined`.
   - `let` is block-scoped, cannot be re-declared in the same scope, and is in the Temporal Dead Zone (TDZ) until declared.
   - `const` is like `let` but its reference cannot be reassigned.

2. **What is the Temporal Dead Zone (TDZ)?**
   - The period between the entering of a scope and the actual variable declaration where accessing `let` or `const` variables throws a ReferenceError.

3. **Can you change the properties of an object declared with `const`?**
   - Yes. `const` prevents reassignment of the variable itself (the reference), but the object's contents can still be mutated.
