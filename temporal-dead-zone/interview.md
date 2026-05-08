# Temporal Dead Zone (TDZ) Interview Questions

1. **What is the Temporal Dead Zone (TDZ)?**
   - The TDZ is the period between entering a block scope and the actual declaration of a `let`, `const`, or `class` variable. During this time, the variable is "hoisted" in the sense that the engine knows it exists, but it is uninitialized and inaccessible. Any attempt to read or write to it will throw a `ReferenceError`.

2. **Why does the TDZ exist in JavaScript?**
   - It was introduced in ES6 to help developers catch bugs. It prevents using variables before they are defined, which leads to more predictable and maintainable code. It also ensures that `const` variables are truly constant from the moment they are accessible.

3. **Which declarations trigger a TDZ?**
   - `let`, `const`, and `class` declarations. `var`, `function` declarations, and `import` statements do NOT have a TDZ (they are initialized as `undefined` or fully hoisted).

4. **Is `typeof` safe to use with variables in the TDZ?**
   - **No.** This is a common trick question. While `typeof` is famously safe for variables that don't exist at all (it returns `"undefined"`), it will throw a `ReferenceError` if used on a variable that is currently in its TDZ.

5. **What is the output of this code?**
   ```javascript
   {
     console.log(name);
     let name = "Richa";
   }
   ```
   - **Output:** `ReferenceError: Cannot access 'name' before initialization`. Even though `name` is hoisted to the top of the block, it is in the TDZ until the `let` line is executed.

6. **How does the TDZ prove that `let` and `const` are actually hoisted?**
   - If they weren't hoisted, the engine would look for the variable in the outer scope (scope chain). For example:
     ```javascript
     let x = 10;
     {
       console.log(x); // If not hoisted, this would be 10
       let x = 20;
     }
     ```
     Because this throws a `ReferenceError` instead of printing `10`, it proves the engine "captured" the inner `x` at the start of the block — which is the definition of hoisting.

7. **Can TDZ occur in function parameters?**
   - Yes. Default parameters are evaluated from left to right. If a parameter on the left tries to access a parameter on the right that hasn't been initialized yet, it will trigger a TDZ error.
     ```javascript
     function test(a = b, b = 5) { // b is in TDZ when a tries to use it
       return a + b;
     }
     test(); // ReferenceError
     ```

8. **When does the TDZ end?**
   - The TDZ ends exactly at the line where the variable is declared and initialized. After that line, the variable is in the "safe zone" and can be used normally.
