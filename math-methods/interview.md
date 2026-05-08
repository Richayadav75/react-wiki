# Math Methods Interview Questions

1. **How do you generate a random integer between 1 and 10?**
   - Use: `Math.floor(Math.random() * 10) + 1`.

2. **What is the difference between `Math.floor()` and `Math.trunc()`?**
   - `Math.floor()` rounds down to the nearest integer.
   - `Math.trunc()` simply removes the decimal part.
   - For negative numbers, they differ: `Math.floor(-1.1)` is `-2`, while `Math.trunc(-1.1)` is `-1`.

3. **How do you find the largest number in an array without using a loop?**
   - Use `Math.max(...array)`. The spread operator expands the array into individual arguments for the `Math.max` method.

4. **What does `toFixed(2)` return?**
   - It returns a **string** representing the number with exactly two decimal places. Since it returns a string, you might need to wrap it in `Number()` if you plan to do more math with it.

5. **How do you get the absolute value of a number?**
   - Use `Math.abs(x)`. It turns negative numbers into positive ones.

6. **Explain `Math.ceil()`.**
   - It always rounds a number **up** to the next largest integer. Even `Math.ceil(1.01)` will return `2`.

7. **Is `Math` a constructor?**
   - No. You cannot do `new Math()`. It is a static object that provides constants and functions.
