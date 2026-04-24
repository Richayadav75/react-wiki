# useCallback Interview Questions

1. **Why do we need useCallback?**
   - To maintain referential equality of functions between renders. This is crucial when passing functions as props to memoized child components (`React.memo`).

2. **Does useCallback make the function faster?**
   - No, it actually adds a small overhead. It makes the **application** faster by preventing unnecessary re-renders of child components.

3. **Relationship between useCallback and useEffect?**
   - If a function is a dependency of `useEffect`, you should wrap it in `useCallback` to prevent the effect from running on every render.
