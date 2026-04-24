# Map and Set Interview Questions

1. **Difference between Map and Object?**
   - Map keys can be any type (objects, functions). Object keys are limited to strings/symbols. Maps maintain insertion order.

2. **When should you use a Set instead of an Array?**
   - When you need to ensure all values are unique and need faster lookups (`has()` is O(1) in Set vs O(n) in Array).

3. **What is a WeakMap?**
   - A Map where keys must be objects and are held "weakly", meaning they don't prevent garbage collection if no other references exist.
