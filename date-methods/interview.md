# Date Methods Interview Questions

1. **How do you get the current timestamp in milliseconds?**
   - `Date.now()` or `new Date().getTime()`.

2. **Why does `getMonth()` return 0 for January?**
   - In JavaScript, months are zero-indexed (0-11).

3. **How do you format a date to a specific locale string?**
   - Use `date.toLocaleDateString('en-US')` or similar locale tags.
