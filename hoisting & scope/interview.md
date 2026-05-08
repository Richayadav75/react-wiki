# Hoisting Interview Questions

1. **What is variable hoisting?**
   - Hoisting is JavaScript's behaviour of processing declarations before execution. During the compilation phase, the engine scans for var declarations, function declarations, and let/const/class declarations and allocates memory. var is initialized as undefined. Function declarations are fully initialized. let/const/class are allocated but kept in TDZ until their line runs.

2. **Difference between var, let, and const hoisting?**
   - `var` hoisted as undefined, function-scoped, accessible before declaration (returns undefined). let/const: hoisted but placed in TDZ, block-scoped, accessing before declaration throws ReferenceError. Both are hoisted — the difference is initialization.

3. **Are functions hoisted?**
   - Function declarations are fully hoisted. Function expressions (e.g., `const x = function() {}`) are not.

4. **Can you call a function before its definition in JavaScript?**
   - `Only for function declarations` — yes. They are fully hoisted (name + body). Function expressions and arrow functions are NOT fully hoisted — they follow var/let/const rules. var function expressions are undefined before their line; const/let ones throw `ReferenceError`


5. **What output does this produce: console.log(x); var x = 5**
   - Output: undefined. var x is hoisted as undefined. The assignment x = 5 happens after the log. If it were let x = 5, it would throw ReferenceError because of TDZ.

6. **What is the output of this code? var x = 1; function test() { console.log(x); var x = 2; } test();**
   - Output: undefined. Inside test(), var x is hoisted to the top of test() as undefined, shadowing the outer x = 1. The log runs before the assignment x = 2, so it prints undefined.

7. **What is the hoisting order when both a var and a function declaration have the same name?**
   - `Function declarations win` — the function gets fully hoisted (name + body). The var declaration still exists but ends up shadowed by the function at the top of the scope. The function is fully ready to call; the var is just `undefined`.

8. **What is the difference between a function declaration and a function expression in terms of hoisting?**
   - Function declaration: fully hoisted — name and body available from the start of scope. Can be called before the definition line. Function expression (var f = function(){}): follows var/let/const hoisting rules. var version is hoisted as undefined; calling it throws TypeError. let/const version is in TDZ; accessing throws ReferenceError.
9. **Are class declarations hoisted?**
   - Yes, class declarations are hoisted — but they are placed in TDZ, just like let/const. You cannot instantiate a class before its declaration line. This differs from function declarations, which are fully hoisted. The TDZ on classes prevents using a class before its full definition is evaluated (important since class bodies run in strict mode).