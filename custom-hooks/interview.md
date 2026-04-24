# Custom Hooks Interview Questions

1. **What is the naming convention for custom hooks?**
   - They must start with the word `use` (e.g., `useAuth`, `useFetch`).

2. **Why extract logic into a custom hook?**
   - For reusability, cleaner components, and separation of concerns.

3. **Do custom hooks share state between components?**
   - No. Every time you use a custom hook, all state and effects inside it are completely isolated to that component instance.
