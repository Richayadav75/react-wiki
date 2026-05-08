# JSON Interview Questions

1. **What does JSON stand for?**
   - JavaScript Object Notation.

2. **What is the difference between `JSON.parse()` and `JSON.stringify()`?**
   - `JSON.parse()`: Converts a JSON string into a usable JavaScript object.
   - `JSON.stringify()`: Converts a JavaScript object into a JSON string.

3. **Can you store functions in JSON?**
   - No. JSON is a data-only format. Functions, `undefined`, and Symbols are ignored during `JSON.stringify()`.

4. **Why are double quotes mandatory in JSON?**
   - JSON follows a strict standard to ensure compatibility across all programming languages (Python, Java, PHP, etc.), most of which require double quotes for string property keys.

5. **How can you deep-clone an object using JSON?**
   - By using `JSON.parse(JSON.stringify(object))`. However, be careful as this will lose any functions, Dates (converted to strings), or `undefined` values stored in the object.

6. **What happens if you try to stringify an object with a circular reference?**
   - It will throw a `TypeError`: "Converting circular structure to JSON".

7. **Are comments allowed in JSON files?**
   - No. Standard JSON does not support comments (`//` or `/* */`). Some specific parsers (like JSONC) allow them, but standard JSON does not.
