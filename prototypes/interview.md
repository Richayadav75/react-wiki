# Prototypes Interview Questions

1. **What is the Prototype Chain?**
   - The mechanism where an object inherits properties/methods from its prototype, which may have its own prototype, and so on until `null`.

2. **What is the difference between `__proto__` and `prototype`?**
   - `prototype` is a property on constructor functions used to set the `__proto__` of new instances. `__proto__` is an internal property of an instance pointing to its prototype.

3. **Why use prototypes instead of defining methods inside the constructor?**
   - Methods on the prototype are shared among all instances, saving memory compared to creating a new copy of the method for every instance.
