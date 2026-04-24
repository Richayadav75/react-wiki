# useRef Interview Questions

1. **Difference between useRef and useState?**
   - `useState` triggers a re-render when the value changes; `useRef` does not. `useRef` returns a mutable object that persists across renders.

2. **When should you use useRef for DOM access?**
   - For managing focus, text selection, media playback, or integrating with third-party DOM libraries.

3. **Can you use useRef to store any value?**
   - Yes, it can store anything (primitive, object, function) that you want to persist without triggering re-renders.
