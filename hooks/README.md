- Category: React Core
- Difficulty: Beginner to Intermediate
- Related: useState, useEffect, useContext, useRef, useMemo, useCallback, useReducer, custom-hooks

### React Hooks — using React features inside function components

Hooks are plain JavaScript functions that start with `use` and let you "hook into" React features — state, lifecycle, context, performance optimisations — from inside a functional component. Introduced in React 16.8, they made class components largely unnecessary.

**Analogy**
Hooks are like power sockets in a hotel room. The room (your function component) does not generate electricity — it just plugs into sockets (`useState`, `useEffect`, etc.) provided by the building (React). Each socket provides a specific capability without you having to wire the building yourself.

---

### 1. Why Hooks Were Introduced

**Theory**
Before hooks, sharing stateful logic between components required Higher-Order Components (HOCs) or render props — both of which added wrapper layers, made debugging hard, and caused "wrapper hell". Class components also split related logic across different lifecycle methods. Hooks solve both problems.

**Working Flow**
![flow-chart](flow-chart.png)

**Example**
```jsx
// BEFORE — class component, logic split across lifecycle methods
class Timer extends React.Component {
  componentDidMount()    { this.interval = setInterval(this.tick, 1000); }
  componentWillUnmount() { clearInterval(this.interval); }
  tick = () => this.setState(s => ({ count: s.count + 1 }));
  render() { return <p>{this.state.count}</p>; }
}

// WITH HOOKS — everything in one place
import { useState, useEffect } from "react";

function Timer() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => setCount(c => c + 1), 1000);
    return () => clearInterval(interval); // cleanup right next to setup
  }, []);

  return <p>{count}</p>;
}
```

**Output**
```
Count ticks up every second: 0 → 1 → 2 → 3 → ...
When component unmounts → interval cleared automatically (no memory leak)
```

---

### 2. Rules of Hooks

**Theory**
React tracks hooks by call order. The two rules of hooks exist to guarantee that call order is the same on every render. Breaking either rule causes bugs that are hard to diagnose.

**Working Flow**
![flow-chart-2](flow-chart-2.png)

**Example**
```jsx
// WRONG — hook inside condition
function Component({ isLoggedIn }) {
  if (isLoggedIn) {
    const [name, setName] = useState(""); // BREAKS rules — conditional hook
  }
  return <div />;
}

// CORRECT — hook at top level, use condition inside
function Component({ isLoggedIn }) {
  const [name, setName] = useState(""); // always called

  if (!isLoggedIn) return <p>Please log in</p>; // condition AFTER hooks
  return <p>Welcome, {name}</p>;
}
```

**Output**
```
WRONG: React throws error or shows inconsistent state across renders
CORRECT: hooks always run in order → React tracks them correctly
```

---

### 3. useState — local state

**Theory**
`useState` gives a functional component memory. It returns the current value and a setter. Calling the setter queues a re-render.

**Working Flow**
![flow-chart-3](flow-chart-3.png)

**Example**
```jsx
import { useState } from "react";

function Toggle() {
  const [on, setOn] = useState(false);
  return (
    <button onClick={() => setOn(prev => !prev)}>
      {on ? "ON" : "OFF"}
    </button>
  );
}
```

**Output**
```
Initial: [OFF]
Click  → [ON]
Click  → [OFF]
```

---

### 4. useEffect — side effects and lifecycle

**Theory**
`useEffect` runs *after* the component renders. It handles anything that reaches outside the component's render cycle: API calls, subscriptions, DOM manipulation, timers. The dependency array controls when it runs.

**Working Flow**
![flow-chart-4](flow-chart-4.png)

**Example**
```jsx
import { useState, useEffect } from "react";

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    let cancelled = false;

    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => {
        if (!cancelled) setUser(data); // guard against stale response
      });

    return () => { cancelled = true; }; // cleanup if userId changes before fetch completes
  }, [userId]); // re-run when userId changes

  if (!user) return <p>Loading...</p>;
  return <p>{user.name}</p>;
}
```

**Output**
```
userId = 1:  "Loading..." → "Alice"
userId = 2:  "Loading..." → "Bob"   (re-fetches when userId prop changes)
Unmount:     cancelled = true (no setState on unmounted component)
```

---

### 5. useContext — consuming global data

**Theory**
`useContext` lets a component read data from a Context without prop drilling. The component subscribes to the nearest matching Provider above it in the tree.

**Working Flow**
![flow-chart-5](flow-chart-5.png)

**Example**
```jsx
import { createContext, useContext, useState } from "react";

const ThemeContext = createContext("light");

function App() {
  const [theme, setTheme] = useState("light");
  return (
    <ThemeContext.Provider value={theme}>
      <button onClick={() => setTheme(t => t === "light" ? "dark" : "light")}>
        Toggle Theme
      </button>
      <Page />
    </ThemeContext.Provider>
  );
}

function Page() {
  return <Card />;  // no theme prop needed here
}

function Card() {
  const theme = useContext(ThemeContext); // reads directly from context
  return (
    <div style={{ background: theme === "dark" ? "#333" : "#fff",
                  color: theme === "dark" ? "#fff" : "#000" }}>
      Current theme: {theme}
    </div>
  );
}
```

**Output**
```
Initial: white background, "Current theme: light"
Toggle:  dark background,  "Current theme: dark"
```

---

### 6. useRef — persistent mutable reference, no re-render

**Theory**
`useRef` returns a mutable object `{ current: value }` that persists across renders without triggering a re-render. Two primary uses: holding a DOM element reference, and storing a value that must persist but should not cause re-renders.

**Working Flow**
![flow-chart-6](flow-chart-6.png)

**Example**
```jsx
import { useRef, useState } from "react";

function StopWatch() {
  const [running, setRunning] = useState(false);
  const [elapsed, setElapsed] = useState(0);
  const intervalRef = useRef(null); // stores interval ID — doesn't need to trigger re-render

  function start() {
    setRunning(true);
    intervalRef.current = setInterval(() => {
      setElapsed(prev => prev + 1);
    }, 1000);
  }

  function stop() {
    clearInterval(intervalRef.current);
    setRunning(false);
  }

  return (
    <div>
      <p>{elapsed}s</p>
      <button onClick={start} disabled={running}>Start</button>
      <button onClick={stop}  disabled={!running}>Stop</button>
    </div>
  );
}
```

**Output**
```
0s  [Start] [Stop-disabled]
Click Start → 1s → 2s → 3s...   [Start-disabled] [Stop]
Click Stop  → timer freezes at 3s
```

---

### 7. useMemo and useCallback — performance optimisation

**Theory**
`useMemo` memoizes a computed value — it recalculates only when dependencies change, not on every render. `useCallback` memoizes a function reference — returns the same function object unless dependencies change. Both prevent unnecessary work but should only be used when there is a measurable performance problem.

**Working Flow**
![flow-chart-7](flow-chart-7.png)

**Example**
```jsx
import { useState, useMemo, useCallback } from "react";

function ProductList({ products }) {
  const [filterText, setFilterText] = useState("");
  const [sortOrder, setSortOrder] = useState("asc");

  // Only re-computed when products, filterText, or sortOrder changes
  const filteredAndSorted = useMemo(() => {
    return products
      .filter(p => p.name.toLowerCase().includes(filterText.toLowerCase()))
      .sort((a, b) => sortOrder === "asc" ? a.price - b.price : b.price - a.price);
  }, [products, filterText, sortOrder]);

  // Same function reference unless sortOrder changes
  const handleSort = useCallback((order) => {
    setSortOrder(order);
  }, []);

  return (
    <div>
      <input value={filterText} onChange={e => setFilterText(e.target.value)} />
      <button onClick={() => handleSort("asc")}>Price Low → High</button>
      <button onClick={() => handleSort("desc")}>Price High → Low</button>
      {filteredAndSorted.map(p => <p key={p.id}>{p.name} — ${p.price}</p>)}
    </div>
  );
}
```

**Output**
```
filterText = "phone":
  iPhone 14 — $999
  Budget Phone — $199

Click "Price Low → High":
  Budget Phone — $199
  iPhone 14 — $999
```

---

### 8. useReducer — complex state with actions

**Theory**
`useReducer` is an alternative to `useState` for state with complex update logic. You define a pure `reducer` function that takes the current state and an action, and returns the next state. Logic is centralised and testable.

**Working Flow**
![flow-chart-8](flow-chart-8.png)

**Example**
```jsx
import { useReducer } from "react";

const initialState = { count: 0, step: 1 };

function reducer(state, action) {
  switch (action.type) {
    case "increment": return { ...state, count: state.count + state.step };
    case "decrement": return { ...state, count: state.count - state.step };
    case "reset":     return initialState;
    case "setStep":   return { ...state, step: action.payload };
    default:          return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <div>
      <p>Count: {state.count} | Step: {state.step}</p>
      <button onClick={() => dispatch({ type: "increment" })}>+</button>
      <button onClick={() => dispatch({ type: "decrement" })}>-</button>
      <button onClick={() => dispatch({ type: "reset" })}>Reset</button>
      <input
        type="number"
        value={state.step}
        onChange={e => dispatch({ type: "setStep", payload: Number(e.target.value) })}
      />
    </div>
  );
}
```

**Output**
```
Count: 0 | Step: 1
Click + → Count: 1
Change Step to 5 → Count: 1 | Step: 5
Click + → Count: 6
Click Reset → Count: 0 | Step: 1
```

---

### Real-World Example — Replacing Class Lifecycle with Hooks

```jsx
// CLASS COMPONENT (old way)
class DataFetcher extends React.Component {
  state = { data: null, loading: true };

  componentDidMount() {          // runs once after mount
    this.fetchData();
  }

  componentDidUpdate(prevProps) {  // runs after every update
    if (prevProps.id !== this.props.id) {
      this.fetchData();            // re-fetch if id changes
    }
  }

  componentWillUnmount() {         // runs before removal
    this.abortController.abort();
  }

  fetchData() { ... }

  render() { return <p>{this.state.data}</p>; }
}

// FUNCTIONAL COMPONENT WITH HOOKS (modern way)
function DataFetcher({ id }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    setLoading(true);
    const controller = new AbortController();

    fetch(`/api/data/${id}`, { signal: controller.signal })
      .then(res => res.json())
      .then(d => { setData(d); setLoading(false); })
      .catch(() => {}); // abort error ignored

    return () => controller.abort(); // cleanup = componentWillUnmount + cancel stale fetch
  }, [id]); // re-run = componentDidUpdate when id changes

  if (loading) return <p>Loading...</p>;
  return <p>{data}</p>;
}
```

**Output**
```
Mount with id=1:       "Loading..." → fetched data shown
Change id prop to 2:   "Loading..." → new data shown (previous fetch aborted)
Unmount:               fetch aborted (no memory leak, no stale setState)

Same behaviour as class component — in ~half the code, with logic grouped together.
```

---

[View Interview Questions](./interview.md)
