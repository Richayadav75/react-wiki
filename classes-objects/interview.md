# Classes & Objects Interview Questions

1. **What is a constructor in a JavaScript class?**
   - It is a special method used for initializing an object's properties. It runs automatically when the `new` keyword is used to create an instance.

2. **How does inheritance work in ES6 classes?**
   - You use the `extends` keyword to create a subclass. You must call `super()` inside the subclass constructor to invoke the parent class constructor and link the `this` context.

3. **What is the `static` keyword used for?**
   - It defines methods or properties that belong to the **class itself**, not to individual instances. You call them directly on the class (e.g., `Math.random()`).

4. **How do you make a property private in a JS class?**
   - Prefix the property name with a hash `#` (e.g., `#balance`). This prevents the property from being accessed outside the class body.

5. **What is the difference between a class and a regular object?**
   - A class is a **blueprint** (template), while an object is an **instance** created from that blueprint. Objects store data, while classes define the structure and behavior.

6. **Explain `super()` in the context of classes.**
   - `super()` is used to call functions on an object's parent. It is most commonly used in a constructor to pass data up to the parent class.

7. **What are Getters and Setters?**
   - They are methods that look like properties. A `get` method computes a value on the fly, and a `set` method validates or processes a value before saving it to a private property.
