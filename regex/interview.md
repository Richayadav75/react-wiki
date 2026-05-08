# Regular Expressions Interview Questions

1. **What is a regular expression?**
   - A pattern used to match, search, or manipulate text in strings. Regex engines scan through a string looking for substrings that fit the defined pattern.

2. **What is the difference between regex literal and RegExp constructor?**
   - Literal `/pattern/flags` is parsed at compile time — use when the pattern is known upfront. `new RegExp(pattern, flags)` is built at runtime — use when the pattern comes from a variable (user input, dynamic search term).

3. **What are common regex flags and what do they do?**
   - `g` (global) — find all matches, not just first. `i` — case-insensitive. `m` — multiline (`^`/`$` match line start/end). `s` — dotall (`.` matches newlines too).

4. **What is the difference between `.test()` and `.match()`?**
   - `regex.test(str)` returns `true`/`false` — use for simple yes/no checks. `str.match(regex)` returns an array of matches (or `null`) — use when you need the actual matched values.

5. **What do `\d`, `\w`, and `\s` match?**
   - `\d` → digit (0-9). `\w` → word character (a-z, A-Z, 0-9, _). `\s` → whitespace (space, tab, newline). Their uppercase versions (`\D`, `\W`, `\S`) match the opposite.

6. **What is the difference between `+` and `*` quantifiers?**
   - `+` matches one or more occurrences (at least one required). `*` matches zero or more (zero is valid). `\d+` requires at least one digit; `\d*` matches even an empty string.

7. **What do `^` and `$` anchors do?**
   - `^` matches the start of the string. `$` matches the end. Together `/^\d{10}$/` means the ENTIRE string must be exactly 10 digits — no characters before or after.

8. **What is a capturing group and how do you access captured values?**
   - Parentheses `()` create a capturing group. The matched substring is accessible in the result array: index 0 is the full match, 1 is the first group, 2 is the second, etc.
   ```javascript
   const m = '2024-05-15'.match(/(\d{4})-(\d{2})-(\d{2})/);
   // m[1] = "2024", m[2] = "05", m[3] = "15"
   ```

9. **How do you validate an email with regex?**
   ```javascript
   const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]{2,}$/;
   emailRegex.test('user@example.com'); // true
   ```

10. **What is the difference between `replace` and `replaceAll` with regex?**
    - `str.replace(/pattern/, replacement)` without the `g` flag replaces only the first match. `str.replace(/pattern/g, replacement)` replaces all. `str.replaceAll('string', replacement)` replaces all occurrences of a plain string.
