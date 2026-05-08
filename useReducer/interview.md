# useReducer Interview Questions

1. **What is useReducer and when is it preferred over useState?**
   - `useReducer` manages state through a reducer function. Prefer it when state has multiple interrelated fields, multiple update types, or when you want state logic isolated and testable outside the component.

2. **What must a reducer function always return?**
   - A new state value. For unknown action types, always return the current `state`. The reducer must be a pure function — no side effects, no direct mutation.
   ```javascript
   function reducer(state, action) {
     switch (action.type) {
       case 'ADD': return [...state, action.payload];
       default:    return state; // always handle unknown actions
     }
   }
   ```

3. **What is the difference between state, action, reducer, and dispatch?**
   - **state**: current value. **action**: `{ type, payload }` describing what happened. **reducer**: `(state, action) => newState`. **dispatch**: function to send actions to the reducer.

4. **Can you use useReducer for global state?**
   - Yes. Combine it with Context API — wrap the app in a Provider that passes down `state` and `dispatch` via context. This is the "mini Redux" pattern — global state without any library.

5. **Why should you never mutate state inside a reducer?**
   - React compares old and new state by reference. Mutating the existing object keeps the same reference → React thinks nothing changed → no re-render. Always return a new object: `{ ...state, count: state.count + 1 }`.

6. **How do you handle async operations with useReducer?**
   - The reducer itself must be synchronous. For async, dispatch a LOADING action before the async call, then dispatch SUCCESS or ERROR with the result:
   ```jsx
   dispatch({ type: 'LOADING' });
   const data = await fetchData();
   dispatch({ type: 'SUCCESS', payload: data });
   ```

7. **What is the payload in an action?**
   - Additional data the reducer needs to compute the next state. The `type` says what to do; `payload` provides the data.
   ```jsx
   dispatch({ type: 'UPDATE_QTY', payload: { id: 2, qty: 5 } });
   ```

8. **How is useReducer different from Redux?**
   - `useReducer` is local (or shared via context). Redux is a global store with middleware, DevTools, time-travel, and performance-optimized subscriptions. Use `useReducer + context` for medium apps, Redux Toolkit for large apps with complex async flows.

9. **Give a scenario where useReducer is clearly better than useState.**
   - A shopping cart with ADD_ITEM, REMOVE_ITEM, UPDATE_QTY, APPLY_COUPON, and CLEAR_CART. Each action needs different logic, and multiple fields (items, total, discount) change together. `useState` would require complex interleaved setters; `useReducer` keeps it clean and testable.

10. **What is the initializer function in useReducer?**
    - The optional third argument to `useReducer`. It receives the second argument (initial value) and computes the actual initial state lazily — useful for expensive initialization or state reset logic.
    ```jsx
    const [state, dispatch] = useReducer(reducer, props.initialCount, init);
    function init(count) { return { count, history: [] }; }
    ```
