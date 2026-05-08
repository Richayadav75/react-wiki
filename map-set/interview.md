# Map & Set Interview Questions

1. **What is the main difference between a Map and an Object?**
   - Objects only allow Strings or Symbols as keys. Maps allow **any** data type as a key, including objects, arrays, and functions. Maps also preserve insertion order and have a built-in `.size` property.

2. **How do you remove duplicates from an array using a Set?**
   - By using: `const unique = [...new Set(myArray)];`.

3. **Does a Set allow duplicate values?**
   - No. A Set is designed specifically to store a collection of **unique** values. Any attempt to add an existing value is ignored.

4. **What are WeakMap and WeakSet?**
   - They are versions of Map/Set where the keys (for WeakMap) or values (for WeakSet) must be **objects**. They hold "weak" references, meaning they don't prevent garbage collection if the object is deleted elsewhere in the code.

5. **How do you check if a value exists in a Set?**
   - Use the `.has(value)` method. It is highly efficient (O(1)) compared to `array.includes()`.

6. **Can you use an object as a key in a Map?**
   - Yes! This is one of the most powerful features of Maps.
     ```javascript
     const user = { name: "Richa" };
     const metadata = new Map();
     metadata.set(user, "Premium User");
     ```

7. **How do you get the number of elements in a Map?**
   - Use the `.size` property. Unlike Objects, you don't need to count keys manually.
