- Category: Hooks
- Difficulty: Beginner
- Related: useReducer, useRef

### What is useState?
`useState` is the most fundamental React Hook. It allows you to add "state" (data that changes) to your functional components.

---

### 1. Core Concept
**Theory**: In React, variables don't trigger UI updates. State does. When you update state, React re-renders the component to show the new value.

**Working Flow**
```text
[ Initial State ] --> [ Render UI ] --> ( User Action ) --> [ SetState ] --> [ Re-Render ]
```

**Key Features**:
- **Reactive**: UI updates automatically when state changes.
- **Persistent**: Remembers values even when the component re-renders.
- **Async**: State updates are processed in batches for performance.

**Example (Counter)**:
```tsx
const [count, setCount] = useState(0);

// Line by line:
// 1. 'count' is the current value (starting at 0).
// 2. 'setCount' is the function we use to change the value.
// 3. To update: setCount(count + 1);
```

---

[View Interview Questions](./interview.md)
