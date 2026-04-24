# Object Methods Interview Questions

1. **How do you clone an object?**
   - Shallow clone: `{...obj}` or `Object.assign({}, obj)`. Deep clone: `JSON.parse(JSON.stringify(obj))` (with limitations) or `structuredClone()`.

2. **Difference between Object.keys(), Object.values(), and Object.entries()?**
   - Keys returns keys, Values returns values, Entries returns an array of [key, value] pairs.

3. **What is the use of Object.freeze()?**
   - It makes an object immutable: you cannot add, remove, or change its properties.
