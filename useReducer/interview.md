# useReducer Interview Questions

1. **When is useReducer preferred over useState?**
   - When the state logic is complex (multiple sub-values), when the next state depends on the previous one, or when you want to pass `dispatch` down instead of multiple callbacks.

2. **What is a Reducer function?**
   - A pure function that takes `state` and `action` and returns a new `state`. It should not have side effects.

3. **Can you use useReducer for global state?**
   - Yes, by combining it with the Context API.
