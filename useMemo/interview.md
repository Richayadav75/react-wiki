# useMemo Interview Questions

1. **What does useMemo do?**
   - Caches the result of a calculation between renders. It only recomputes when a listed dependency changes — returning the cached result on every other render.

2. **What is the difference between useMemo and useCallback?**
   - `useMemo` memoizes the **return value** of a function (a computed result like an array or object). `useCallback` memoizes the **function itself** (useful when passing stable callbacks as props).

3. **What is the difference between useMemo and React.memo?**
   - `useMemo` memoizes a **value** inside a component. `React.memo` memoizes an entire **component** to prevent re-renders when its props haven't changed.

4. **When should you use useMemo?**
   - When a calculation is genuinely expensive (filtering thousands of items, complex sorting) and its inputs don't change on every render. Also when you need a stable object/array reference to prevent `React.memo` children from re-rendering.

5. **When should you NOT use useMemo?**
   - For cheap calculations. `useMemo` itself has overhead — storing the value, comparing deps on every render. For simple string concatenation or arithmetic, just compute directly.
   ```jsx
   const doubled = count * 2;          // ✅ no useMemo needed
   const greeting = `Hello, ${name}!`; // ✅ no useMemo needed
   ```

6. **What is referential stability and why does useMemo help with it?**
   - In JavaScript `{} !== {}` — two separate object literals are different references even if their content is identical. Objects/arrays created inside a component are new references every render. `useMemo` returns the same reference unless deps change, making it safe to pass to `React.memo` children.

7. **What happens if you omit the dependency array in useMemo?**
   - The function runs on every render, caching nothing — defeating the whole purpose. Always include the dependency array.

8. **How does useMemo compare its dependencies?**
   - Using `Object.is` (strict equality). Primitive values are compared by value; objects by reference.

9. **Can useMemo be used outside a React component?**
   - No. Like all hooks, `useMemo` must be called at the top level of a React functional component or custom hook.

10. **What would happen if you removed useMemo from a working optimization?**
    - The calculation would run on every render. If the calculation is cheap, no visible effect. If it's expensive (filtering 10k items), you'd notice sluggish UI on every keystroke or state change.
