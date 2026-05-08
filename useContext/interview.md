# useContext Interview Questions

1. **What is useContext and what problem does it solve?**
   - `useContext` reads a value from a React context inside a functional component, eliminating prop drilling — passing data through many layers of components that don't use it themselves.

2. **What are the three steps to use Context?**
   - (1) `createContext(defaultValue)` — create the context. (2) `<Context.Provider value={...}>` — provide the value. (3) `useContext(Context)` — consume the value in any descendant.

3. **What is the default value in createContext?**
   - Used only when a component calls `useContext` without being wrapped in a matching Provider. Usually `null` or a sensible fallback — useful for testing components in isolation.

4. **What happens when the context value changes?**
   - Every component that calls `useContext(MyContext)` re-renders automatically, regardless of which part of the value they use. This is why splitting unrelated data into separate contexts matters for performance.

5. **How do you make context dynamic (changeable at runtime)?**
   - Pair the Provider with `useState` or `useReducer`. Pass both the state value and the setter as the context value.
   ```jsx
   const [theme, setTheme] = useState('light');
   <ThemeContext.Provider value={{ theme, setTheme }}>
   ```

6. **How do you prevent unnecessary re-renders from context?**
   - Wrap the context value in `useMemo` so a new object isn't created on every parent render. Also split context by concern — components only subscribe to contexts they actually need.

7. **Does Context replace Redux?**
   - For simple to medium apps, yes. Redux provides better DevTools, middleware (for complex async), and optimized subscriptions for large state trees. `useContext + useReducer` is the lightweight alternative.

8. **What is the common performance pitfall with useContext?**
   - Putting everything in one context. If user info and theme are in the same context, updating the theme re-renders every component using the context — even those that only care about user info. Solution: split into `ThemeContext` and `UserContext`.

9. **How do you throw an error if a component uses useContext outside its Provider?**
   ```jsx
   function useAuth() {
     const ctx = useContext(AuthContext);
     if (!ctx) throw new Error('useAuth must be inside AuthProvider');
     return ctx;
   }
   ```

10. **Can you have multiple contexts in one app?**
    - Yes. Each concern gets its own context: `ThemeContext`, `AuthContext`, `CartContext`. A component wraps itself in only the contexts it needs, and only re-renders when those specific contexts change.
