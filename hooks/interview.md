# React Hooks Interview Questions

---

**1. What are React Hooks and why were they introduced?**

Hooks are functions that let functional components use React features like state and lifecycle methods. They were introduced in React 16.8 to eliminate the need for class components, reduce wrapper hell caused by HOCs/render props, and allow grouping of related logic in one place instead of splitting it across lifecycle methods.

```jsx
// Before: logic split across lifecycle methods in a class
componentDidMount()    { this.startTimer(); }
componentWillUnmount() { this.stopTimer(); }

// After: same logic grouped in one useEffect
useEffect(() => {
  const id = startTimer();
  return () => stopTimer(id); // setup and cleanup together
}, []);
```

---

**2. What are the two Rules of Hooks and why do they exist?**

1. Only call hooks at the top level (not inside loops, conditions, or nested functions).
2. Only call hooks from React functional components or custom hooks.

React tracks hooks by their call order on each render. If a hook is skipped (due to a condition) or called a different number of times, React loses track of which state belongs to which hook call.

```jsx
// WRONG — conditional hook breaks ordering
function Bad({ show }) {
  if (show) {
    const [x, setX] = useState(0); // skipped when show=false → order breaks
  }
}

// CORRECT — hooks always at top
function Good({ show }) {
  const [x, setX] = useState(0); // always called
  if (!show) return null;
}
```

---

**3. What does the dependency array in useEffect do?**

It tells React when to re-run the effect:
- `[]` — run once after mount (like `componentDidMount`)
- `[a, b]` — run after mount and whenever `a` or `b` changes
- Omitted — run after every render

```jsx
useEffect(() => { console.log("mount only"); },  []);      // once
useEffect(() => { console.log("id changed"); },  [id]);    // on id change
useEffect(() => { console.log("every render"); });          // every render
```

---

**4. What is the difference between useState and useRef?**

Both persist a value across renders. The key difference: updating `useState` triggers a re-render; updating `useRef.current` does not.

```jsx
const [count, setCount] = useState(0); // changing triggers re-render
const timerRef = useRef(null);          // changing does NOT trigger re-render

// Use useState for values displayed in the UI
// Use useRef for internal implementation details (interval IDs, DOM nodes, previous values)
```

---

**5. When should you use useMemo vs useCallback?**

- `useMemo` — memoizes a *computed value* (the result of calling a function). Use when a calculation is expensive and runs on every render unnecessarily.
- `useCallback` — memoizes a *function reference*. Use when a child component wrapped in `React.memo` receives a callback — otherwise a new function reference on each render causes the child to re-render anyway.

```jsx
// useMemo — cache the result of filtering a large list
const filtered = useMemo(() =>
  bigList.filter(item => item.active), [bigList]);

// useCallback — stable reference for a memoized child
const handleClick = useCallback(() => {
  doSomething(id);
}, [id]);

return <MemoizedChild onClick={handleClick} />;
```

---

**6. What is the cleanup function in useEffect and when does it run?**

The cleanup function is the function returned from the effect callback. It runs in two situations:
1. Just before the component unmounts.
2. Before the effect runs again (when dependencies change).

```jsx
useEffect(() => {
  const subscription = subscribe(userId);

  return () => {
    subscription.unsubscribe(); // runs when userId changes (before re-subscribe)
                                // AND when component unmounts
  };
}, [userId]);
```

---

**7. What is a custom hook and how do you create one?**

A custom hook is a JavaScript function whose name starts with `use` and that calls other hooks inside it. It lets you extract and share stateful logic between components without adding wrapper components.

```jsx
// Custom hook — extracts fetch logic
function useFetch(url) {
  const [data, setData]     = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError]   = useState(null);

  useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then(d => { setData(d); setLoading(false); })
      .catch(e => { setError(e); setLoading(false); });
  }, [url]);

  return { data, loading, error };
}

// Usage in any component
function UserProfile({ id }) {
  const { data, loading } = useFetch(`/api/users/${id}`);
  if (loading) return <p>Loading...</p>;
  return <p>{data.name}</p>;
}
```

---

**8. What is useReducer and when would you choose it over useState?**

`useReducer` manages state through a pure reducer function and dispatched actions. Choose it over `useState` when:
- Multiple state values are related and updated together.
- The update logic is complex (multiple action types).
- You want the update logic to be testable in isolation.

```jsx
function reducer(state, action) {
  switch (action.type) {
    case "increment": return { ...state, count: state.count + 1 };
    case "reset":     return { count: 0, step: 1 };
    default:          return state;
  }
}

const [state, dispatch] = useReducer(reducer, { count: 0, step: 1 });
dispatch({ type: "increment" }); // → state.count becomes 1
```

---

**9. Why should you not call hooks inside a regular JavaScript function?**

Regular functions are not managed by React — they have no concept of component identity or render cycle. Hooks rely on React's internal fiber tracking, which only works inside React function components and custom hooks (which React recognizes because their name starts with `use` and they are called from within the React rendering context).

```jsx
// WRONG — hook inside a utility function
function formatUser(user) {
  const [cached, setCached] = useState(user); // React error — no component context
  return cached;
}

// CORRECT — if you need state with the logic, make it a custom hook
function useCachedUser(user) {
  const [cached, setCached] = useState(user); // works — React sees this as a hook
  return cached;
}
```

---

**10. What happens if you call useEffect without a dependency array vs with an empty array?**

Without a dependency array — the effect runs after every single render. This can cause infinite loops if you update state inside it.

With an empty array `[]` — the effect runs only once after the first render. The cleanup function runs when the component unmounts.

```jsx
// Runs after every render — careful with state updates inside!
useEffect(() => {
  document.title = `Count: ${count}`;
}); // no dep array

// Runs once after mount
useEffect(() => {
  fetchInitialData();
}, []); // empty array

// Common bug — infinite loop
useEffect(() => {
  setCount(count + 1); // state update → re-render → effect runs → state update → ...
}); // no dep array = infinite loop!
```
