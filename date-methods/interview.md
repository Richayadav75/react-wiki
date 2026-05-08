# Date Methods Interview Questions

1. **How do months work in the JavaScript `Date` object?**
   - They are zero-indexed. This means January is `0`, February is `1`, and December is `11`.

2. **What is the difference between `getDate()` and `getDay()`?**
   - `getDate()` returns the day of the month (1-31).
   - `getDay()` returns the day of the week (0-6), where 0 is Sunday.

3. **What is the Unix Epoch?**
   - It is January 1, 1970, UTC. JavaScript dates are stored as the number of milliseconds elapsed since this specific moment.

4. **How do you get the current timestamp in milliseconds?**
   - Use `Date.now()` (static method) or `new Date().getTime()`.

5. **How can you add 7 days to a current date?**
   - ```javascript
     const date = new Date();
     date.setDate(date.getDate() + 7);
     ```

6. **What does `toISOString()` do?**
   - It returns a string in the simplified extended ISO format (ISO 8601), which is always 24 or 27 characters long (`YYYY-MM-DDTHH:mm:ss.sssZ`). It is the standard format for sending dates to a server.

7. **How do you format a date to a human-readable local string?**
   - Use `toLocaleDateString()`. You can pass a locale (e.g., `'en-GB'`) and an options object to customize the output.
