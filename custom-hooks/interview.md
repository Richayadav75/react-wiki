# Custom Hooks Interview Questions

1. **What is a Custom Hook?**
   - It is a JavaScript function that starts with `use` and calls other React hooks. It is used to share stateful logic between different components.

2. **Why must a custom hook's name start with `use`?**
   - This is a convention required by the React team. It allows the ESLint plugin for React Hooks to identify that the function should follow the "Rules of Hooks" (e.g., no conditional calls).

3. **Does sharing a custom hook share the state between components?**
   - **No.** Every time a component calls a custom hook, it gets its own independent copy of the state and effects inside that hook. It is a way to reuse **logic**, not data.

4. **What can a custom hook return?**
   - Anything! It can return an array (like `useState`), an object (like `useFetch`), a single value, or even nothing at all.

5. **When should you create a custom hook?**
   - When you find yourself repeating the same `useEffect` or `useState` logic across multiple components (e.g., handling form inputs, fetching data, or syncing with localStorage).

6. **Can a custom hook call another custom hook?**
   - Yes. Hooks can be composed just like regular functions.

7. **How do custom hooks help with testing?**
   - They allow you to test complex stateful logic in isolation without needing to mount a full UI component, often using libraries like `@testing-library/react-hooks`.
 Riverside.
 Riverside.
