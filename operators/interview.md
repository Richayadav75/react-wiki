# Operators Interview Questions

1. **What is the difference between `==` and `===`?**
   - `==` (Abstract Equality) performs type coercion before comparison.
   - `===` (Strict Equality) compares both value and type without coercion.

2. **What is "Type Coercion" in JavaScript?**
   - It's the automatic conversion of values from one data type to another (e.g., `5 + "5"` becomes `"55"`).

3. **What is the use of the Nullish Coalescing Operator (`??`)?**
   - It returns the right-hand side operand when the left-hand side is `null` or `undefined`, unlike `||` which triggers for any falsy value (like `0` or `""`).
