# useRef Interview Questions

1. **What is useRef and what makes it different from useState?**
   - `useRef` returns a mutable object `{ current: initialValue }`. Unlike `useState`, changing `.current` does NOT trigger a re-render. It's used when you need to persist a value across renders without the component knowing about it.

2. **What are the two main use cases for useRef?**
   - (1) **DOM access** — attaching to a JSX element via `ref` attribute to call native DOM methods like `.focus()`, `.scrollIntoView()`. (2) **Persisting values** — storing timer IDs, previous values, or any mutable data that shouldn't trigger re-renders.

3. **How do you focus an input using useRef?**
   ```jsx
   const inputRef = useRef(null);
   // Attach: <input ref={inputRef} />
   // Focus: inputRef.current.focus();
   ```

4. **Why would you store a setInterval ID in a ref instead of state?**
   - Storing it in state would trigger a re-render when the timer starts/stops, which is unnecessary. A ref stores it silently so you can cancel it in a cleanup without any UI side effect.

5. **What is the "previous value" pattern with useRef?**
   ```jsx
   const prevRef = useRef();
   useEffect(() => {
     prevRef.current = value; // update after each render
   });
   // prevRef.current holds the value from the previous render
   ```

6. **Can you pass a ref to a custom component?**
   - Not directly — refs don't work on custom components by default. You must use `React.forwardRef` to forward the ref to a specific DOM element inside the custom component.

7. **What does forwardRef do?**
   - It wraps a functional component to accept a `ref` argument and forward it to a DOM element inside. This lets parents access a child's DOM node.

8. **When should you use useRef vs useState for a counter?**
   - If the counter value needs to display in the UI → `useState`. If it's only used internally (tracking render count, for debugging) and shouldn't cause UI updates → `useRef`.

9. **Does changing ref.current cause a re-render?**
   - No. `ref.current` is a plain mutable property — React doesn't track it. This is both the feature (no wasted renders) and the limitation (you can't use it to drive UI updates).

10. **What is the initial value of a useRef if you pass null?**
    - `ref.current` is `null` until React attaches the DOM element to it (after the component mounts). Always check `ref.current` before calling methods on it.
