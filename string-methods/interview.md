# String Methods Interview Questions

1. **Difference between slice() and substring()?**
   - `slice()` can take negative indices (counting from the end). `substring()` treats negative values as 0.

2. **How do you check if a string contains a substring?**
   - Using `includes()`, `indexOf() !== -1`, or `search()` with regex.

3. **Are strings mutable in JavaScript?**
   - No, strings are immutable. Every string method returns a new string instead of modifying the original.
