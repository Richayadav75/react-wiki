# React.memo Interview Questions

1. **What is the default comparison for React.memo?**
   - A shallow comparison of props.

2. **How do you handle object or function props with React.memo?**
   - Use `useMemo` for objects and `useCallback` for functions in the parent component to keep references stable.

3. **Can you provide a custom comparison function?**
   - Yes, as a second argument: `React.memo(Component, (prev, next) => { ... })`.
