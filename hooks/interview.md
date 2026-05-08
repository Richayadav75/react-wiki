# React Hooks Interview Questions

1. **What are React Hooks?**
   - They are functions that allow functional components to use state, lifecycle methods, and other React features that were previously only available in class components.

2. **What are the two most important rules of Hooks?**
   1. Only call Hooks at the top level (not inside loops or conditions).
   2. Only call Hooks from React functional components or custom hooks.

3. **Why shouldn't you call Hooks inside an `if` statement?**
   - Because React relies on the **order** in which Hooks are called to keep track of state. If a Hook call is skipped due to a condition, the internal order is broken, leading to bugs.

4. **What is the difference between `useState` and `useRef`?**
   - `useState` triggers a **re-render** when the value changes.
   - `useRef` persists the value across renders but does **not** trigger a re-render when it changes.

5. **When would you use `useMemo`?**
   - When you have an expensive calculation that you want to avoid repeating on every render. It only recalculates the value when its dependencies change.

6. **What is a Custom Hook?**
   - It is a function that starts with `use` and calls other Hooks. It allows you to extract and reuse stateful logic between different components.

7. **How do Hooks improve code readability?**
   - They allow you to group related logic together (e.g., all fetch logic in one `useEffect`) rather than splitting it across separate lifecycle methods like `componentDidMount` and `componentWillUnmount`.
 Riverside.
 Riverside.
