- Category: State Management
- Difficulty: Intermediate
- Related: useState

### What is useReducer?
`useReducer` is an alternative to `useState` for managing complex state logic that involves multiple sub-values or depends on the previous state.

---

### 1. State Management Logic
**Theory**: It uses a "reducer" function that takes the current state and an "action", then returns the new state.

**Working Flow**
```text
[ Dispatch Action ] --> [ Reducer Function ] --> [ New State ]
```

**Example**:
```tsx
const [state, dispatch] = useReducer(reducer, { count: 0 });

// Triggering an update:
dispatch({ type: 'increment' });
```

---

[View Interview Questions](./interview.md)
