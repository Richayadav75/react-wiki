# State Interview Questions

---

**1. What is state in React and how is it different from a regular variable?**

State is React-managed data that, when changed via the setter function, triggers a re-render. A regular variable change is invisible to React — the UI will not update.

```jsx
// Regular variable — React does NOT track this
let count = 0;
function handleClick() { count++; } // UI stays at 0 forever

// State — React tracks this and re-renders on change
const [count, setCount] = useState(0);
function handleClick() { setCount(count + 1); } // UI updates
```

---

**2. How do you declare and use state in a functional component?**

Using the `useState` hook. It returns a tuple: `[currentValue, setterFunction]`. The argument to `useState` is the initial value, applied only on the first render.

```jsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0); // initial = 0

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

---

**3. Why should you never mutate state directly?**

Direct mutation keeps the same object reference. React compares references to detect changes — if the reference is the same, React skips the re-render, and the UI stays stale.

```jsx
// WRONG — no re-render triggered
const [user, setUser] = useState({ name: "Alice" });
user.name = "Bob";   // mutates in place
setUser(user);       // same reference → React may skip re-render

// CORRECT — new object reference → guaranteed re-render
setUser({ ...user, name: "Bob" });
```

---

**4. What is the stale closure problem and how does functional update form solve it?**

When multiple state updates happen in one event handler, each closure may capture the same old value of the state variable. Passing a function to the setter ensures each update receives the latest queued value.

```jsx
// WRONG — count captured from closure, all three see count = 0
setCount(count + 1);
setCount(count + 1);
setCount(count + 1);
// result: count = 1

// CORRECT — functional form chains updates
setCount(prev => prev + 1);
setCount(prev => prev + 1);
setCount(prev => prev + 1);
// result: count = 3
```

---

**5. How do you update one field in an object stored in state?**

Spread the existing object to preserve all other fields, then override only the one that changed. Never mutate the object directly.

```jsx
const [user, setUser] = useState({ name: "Alice", email: "", age: 28 });

// Update only the name — email and age stay the same
setUser(prev => ({ ...prev, name: "Bob" }));

// With a generic handler for forms:
function handleChange(e) {
  const { name, value } = e.target;
  setUser(prev => ({ ...prev, [name]: value }));
}
```

---

**6. What are the correct immutable patterns for array state?**

Never call `push`, `pop`, `splice`, or `sort` on the state array. Produce a new array instead.

```jsx
const [todos, setTodos] = useState([]);

// Add
setTodos(prev => [...prev, newTodo]);

// Remove by id
setTodos(prev => prev.filter(t => t.id !== targetId));

// Update one item
setTodos(prev =>
  prev.map(t => t.id === targetId ? { ...t, done: true } : t)
);
```

---

**7. Is setState synchronous or asynchronous?**

Asynchronous. React batches state updates and applies them before the next render. Reading the state variable immediately after calling the setter still gives the old value.

```jsx
const [count, setCount] = useState(0);

function handleClick() {
  setCount(1);
  console.log(count); // still 0 — the re-render hasn't happened yet
}
// count will be 1 on the NEXT render
```

---

**8. What does "lifting state up" mean?**

When two sibling components need to share data, the state is moved to their closest common parent. The parent owns the state and passes it down as props to both children — this keeps a single source of truth.

```jsx
function App() {
  const [query, setQuery] = useState("");
  return (
    <>
      <SearchBox value={query} onChange={setQuery} />
      <ResultsList query={query} />
    </>
  );
}
// Both SearchBox and ResultsList are always in sync because
// they both read from the same piece of state in App.
```

---

**9. What data should NOT go into state?**

- Values that can be computed from existing state or props (derive them instead).
- DOM element references (use `useRef`).
- Data that does not affect the rendered output.
- Props passed from the parent (reading them directly is enough).

```jsx
// BAD — fullName in state is redundant
const [firstName, setFirstName] = useState("Alice");
const [fullName,  setFullName]  = useState("Alice Smith"); // gets stale

// GOOD — derive fullName
const [firstName, setFirstName] = useState("Alice");
const [lastName,  setLastName]  = useState("Smith");
const fullName = `${firstName} ${lastName}`; // always fresh
```

---

**10. When would you use useReducer instead of useState?**

`useReducer` is better when:
- State has complex update logic (multiple related fields).
- Multiple different actions can update the same state.
- The next state depends on both the current state and the action type.

```jsx
const initialState = { count: 0, step: 1 };

function reducer(state, action) {
  switch (action.type) {
    case "increment": return { ...state, count: state.count + state.step };
    case "decrement": return { ...state, count: state.count - state.step };
    case "setStep":   return { ...state, step: action.payload };
    default:          return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <div>
      <p>Count: {state.count} (step: {state.step})</p>
      <button onClick={() => dispatch({ type: "increment" })}>+</button>
      <button onClick={() => dispatch({ type: "setStep", payload: 5 })}>Step=5</button>
    </div>
  );
}
```
