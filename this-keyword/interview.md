# "this" Keyword Interview Questions

1. **What does the `this` keyword refer to in JavaScript?**
   - It refers to the object that is currently executing the function. Its value is determined by how the function is called, not where it is defined.

2. **Explain the difference between `call`, `apply`, and `bind`.**
   - `call`: Invokes the function immediately, passing arguments individually.
   - `apply`: Invokes the function immediately, passing arguments as an array.
   - `bind`: Returns a new function with `this` permanently bound, without invoking it immediately.

3. **How does `this` behave in arrow functions?**
   - Arrow functions do not have their own `this`. They inherit `this` from the outer lexical scope (the parent). This is why they are great for callbacks inside methods.

4. **What happens to `this` in a regular function in strict mode?**
   - In strict mode, if a function is called without a clear owner (like a global call), `this` will be `undefined` instead of the `window` object.

5. **What is the output of the following code?**
   ```javascript
   const obj = {
     name: "Richa",
     greet: function() {
       setTimeout(function() {
         console.log(this.name);
       }, 100);
     }
   };
   obj.greet();
   ```
   - **Output**: `undefined` (or an empty string in browsers).
   - **Reason**: The callback inside `setTimeout` is a regular function, and it loses the binding to `obj`. In a regular function, `this` defaults to `window`. To fix it, use an arrow function.

6. **What is "Implicit Binding"?**
   - It is when a function is called as a method of an object (e.g., `user.getName()`). In this case, `this` is implicitly bound to the object before the dot.

7. **What is the "new" binding?**
   - When a function is invoked with the `new` keyword, `this` inside that function is bound to the brand-new object being created.
