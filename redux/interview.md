# Redux Interview Questions

1. **What are the three core principles of Redux?**
   - **Single source of truth**: The state of your whole application is stored in an object tree within a single store.
   - **State is read-only**: The only way to change the state is to emit an action.
   - **Changes are made with pure functions**: To specify how the state tree is transformed by actions, you write pure reducers.

2. **What is an Action in Redux?**
   - An action is a plain JavaScript object that describes a change. It must have a `type` property and can optionally have a `payload` containing data.

3. **What is a Reducer?**
   - A reducer is a pure function that takes the previous state and an action, and returns the next state. It must not modify the existing state; instead, it returns a new object.

4. **What is Redux Toolkit (RTK)?**
   - RTK is the official, opinionated, battery-included toolset for efficient Redux development. It simplifies tasks like store setup, creating reducers, and handling immutable state updates.

5. **How is Redux different from the Context API?**
   - **Context API**: Great for sharing low-frequency updates (like theme or auth) across a component tree. It can lead to unnecessary re-renders if used for complex state.
   - **Redux**: Designed for high-frequency updates and complex logic. It provides powerful debugging tools (DevTools) and a more structured data flow.

6. **What is "Middleware" in Redux?**
   - Middleware provides a third-party extension point between dispatching an action and the moment it reaches the reducer. It is used for logging, crash reporting, and handling asynchronous requests (e.g., Redux Thunk or Redux Saga).

7. **What is a "Store Selector"?**
   - A selector is a function that extracts specific pieces of state from the store. Libraries like `reselect` can be used to memoize these functions for better performance.
 Riverside.
 Riverside.
