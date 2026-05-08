# String Methods Interview Questions

1. **Are strings mutable in JavaScript?**
   - No, strings are immutable. This means that any method that "modifies" a string actually returns a new string. The original remains unchanged.

2. **What is the difference between `slice()` and `substring()`?**
   - `slice(start, end)` can accept negative indices (which count from the end).
   - `substring(start, end)` treats negative values as 0 and will swap `start` and `end` if the start is greater than the end.

3. **How do you convert a string to an array?**
   - Using the `split()` method. For example, `"a,b,c".split(",")` returns `["a", "b", "c"]`.

4. **How do you remove extra spaces from the start and end of a string?**
   - Using the `trim()` method. There are also `trimStart()` and `trimEnd()` for more specific needs.

5. **Explain `padStart()` and `padEnd()`.**
   - These methods pad the current string with another string until it reaches a given length. Useful for formatting (like credit card masking or adding leading zeros).

6. **How can you check if a string contains a specific substring?**
   - Use `includes()` for a boolean result, or `indexOf()` if you need the position.

7. **How do you replace all occurrences of a word in a string?**
   - Before ES2021, you had to use a global regex: `str.replace(/word/g, "new")`.
   - Now, you can use the more readable `str.replaceAll("word", "new")`.
