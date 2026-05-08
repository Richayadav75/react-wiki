# Object Methods Interview Questions

1. **How do you get an array of all keys in an object?**
   - Use `Object.keys(obj)`.

2. **What is the difference between `Object.freeze()` and `Object.seal()`?**
   - `Object.seal()`: Prevents adding or deleting properties, but allows **modifying** existing ones.
   - `Object.freeze()`: Prevents adding, deleting, AND modifying properties. It makes the object completely immutable.

3. **How do you convert an object into an array of its key-value pairs?**
   - Use `Object.entries(obj)`. This is very useful for looping through objects with `forEach` or `map`.

4. **What does `Object.fromEntries()` do?**
   - It performs the reverse of `Object.entries()`. It takes an array of `[key, value]` pairs and turns it back into an object.

5. **How can you merge two objects?**
   - Use `Object.assign(target, source1, source2)` or the spread operator `{ ...obj1, ...obj2 }`. If there are duplicate keys, the last one in the list wins.

6. **Does `Object.assign()` perform a deep or shallow copy?**
   - It performs a **shallow copy**. If the source object contains nested objects, only the references to those objects are copied, not the objects themselves.

7. **How do you check if an object has a specific property?**
   - Use `obj.hasOwnProperty("propName")` or the `in` operator (`"propName" in obj`).
