# useState Interview Questions

1. **Why is useState asynchronous?**
   - React batches state updates for performance. Updating state immediately would trigger multiple re-renders. By batching, React ensures only one re-render happens per event loop tick.

2. **How do you update state based on previous state?**
   - Always use the functional update pattern: `setCount(prev => prev + 1)`. This ensures you are working with the most up-to-date state value.

3. **What happens if you update state with the same value?**
   - React will bail out and not re-render the component or fire effects (using Object.is comparison).
