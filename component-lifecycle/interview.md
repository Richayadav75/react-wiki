# Component Lifecycle Interview Questions

1. **What are the three main phases of a React component's lifecycle?**
   - **Mounting**: Adding the component to the DOM.
   - **Updating**: Re-rendering due to state or prop changes.
   - **Unmounting**: Removing the component from the DOM.

2. **How do you replicate `componentDidMount` in a functional component?**
   - By using `useEffect` with an empty dependency array: `useEffect(() => { ... }, [])`.

3. **What is the purpose of the cleanup function in `useEffect`?**
   - It is used to clear side effects like timers, event listeners, or subscriptions. It prevents **memory leaks** by ensuring resources are freed when the component is unmounted or before the effect runs again.

4. **When does the cleanup function run?**
   - It runs in two cases:
     1. Just before the component unmounts.
     2. Just before the effect runs again (if dependencies have changed).

5. **What is the difference between `useEffect` and `useLayoutEffect`?**
   - `useEffect` runs **asynchronously** after the browser has painted the screen.
   - `useLayoutEffect` runs **synchronously** after DOM mutations but **before** the browser paints. It's used for measuring the DOM to avoid visual flickers.

6. **What happens if you omit the dependency array in `useEffect`?**
   - The effect will run after **every single render**, which can lead to performance issues or infinite loops if you update state inside that effect.

7. **How do you simulate `componentDidUpdate` only for a specific value?**
   - By including that value in the dependency array: `useEffect(() => { ... }, [myValue])`.
 Riverside.
 Riverside.
