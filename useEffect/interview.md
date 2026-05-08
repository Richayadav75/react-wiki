# useEffect Interview Questions

1. **What is useEffect used for?**
   - Performing side effects in functional components — things that happen outside the render: fetching data, subscribing to events, setting up timers, manipulating the DOM, or syncing with external systems.

2. **What are the three forms of the dependency array?**
   - **No array** → runs after every render. **`[]`** → runs once after first render (mount). **`[dep1, dep2]`** → runs after first render and whenever any listed dep changes.

3. **What is the cleanup function and when does it run?**
   - The function returned from useEffect. It runs: (1) when the component unmounts, and (2) before the effect re-runs (when deps change). Used to clear timers, remove event listeners, cancel fetch requests.
   ```jsx
   useEffect(() => {
     const id = setInterval(tick, 1000);
     return () => clearInterval(id); // cleanup
   }, []);
   ```

4. **What happens if you set state inside useEffect without a dep array?**
   - Infinite loop: useEffect runs → sets state → re-render → useEffect runs → sets state → ♾️. Always include deps.

5. **Why can't useEffect itself be async?**
   - `useEffect` must return either nothing or a cleanup function. An `async` function always returns a Promise — React doesn't know what to do with a Promise as a cleanup function. Fix: define an `async` function inside the effect and call it.
   ```jsx
   useEffect(() => {
     async function load() {
       const data = await fetchData();
       setData(data);
     }
     load();
   }, []);
   ```

6. **What is a stale closure in useEffect?**
   - When a variable is used inside useEffect but not listed in deps, the effect "closes over" the old value and never sees updates. Fix: add the variable to the dep array.

7. **How do you cancel a fetch request in useEffect?**
   - Using `AbortController`. Pass the signal to fetch, and abort in the cleanup function:
   ```jsx
   useEffect(() => {
     const controller = new AbortController();
     fetch(url, { signal: controller.signal }).then(...)
     return () => controller.abort();
   }, [url]);
   ```

8. **How does useEffect replicate componentDidMount and componentWillUnmount?**
   ```jsx
   useEffect(() => {
     // componentDidMount
     return () => {
       // componentWillUnmount
     };
   }, []); // empty deps = only on mount/unmount
   ```

9. **What is the difference between useEffect and useLayoutEffect?**
   - `useEffect` fires asynchronously after the browser has painted. `useLayoutEffect` fires synchronously before paint — use it for DOM measurements or mutations that must happen before the user sees the screen.

10. **Why does the dependency array need to be exhaustive?**
    - If a dep is missing, the effect reads a stale value. ESLint's `exhaustive-deps` rule warns about this. If adding a dep causes an unwanted effect re-run, the real fix is restructuring the logic — not removing the dep.
