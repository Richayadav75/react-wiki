# Component Lifecycle Interview Questions

1. **What are the three main phases of a component?**
   - Mount, Update, and Unmount.

2. **Which hook replaces componentWillUnmount?**
   - The cleanup function returned by `useEffect(() => { ... return () => cleanup }, [])`.

3. **What is the difference between useEffect and useLayoutEffect?**
   - `useEffect` runs asynchronously after the paint. `useLayoutEffect` runs synchronously after DOM mutations but before the browser paints.
