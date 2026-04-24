# useEffect Interview Questions

1. **What happens if you omit the dependency array?**
   - The effect runs after every single render.

2. **How do you mimic componentDidMount with useEffect?**
   - Pass an empty dependency array `[]`.

3. **What is the purpose of the cleanup function?**
   - It runs before the component unmounts and before the effect re-runs (if dependencies changed). It's used for clearing timers, subscriptions, or global event listeners.
