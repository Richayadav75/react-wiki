# Math Methods Interview Questions

1. **How do you generate a random number between a specific range (min, max)?**
   - `Math.floor(Math.random() * (max - min + 1)) + min`.

2. **What is the difference between `Math.floor()` and `Math.trunc()`?**
   - `Math.floor()` rounds down towards negative infinity.
   - `Math.trunc()` simply removes the fractional part, rounding towards zero.

3. **How do you find the maximum value in an array using Math?**
   - `Math.max(...array)` using the spread operator.
