- Category: React Basics
- Difficulty: Beginner
- Related: hooks, component-lifecycle, useState, useEffect, error-boundaries

### Class vs Function Components — The Full Comparison
React has two ways to define a component. **Class components** are the original form from React's early days, using ES6 class syntax, `this`, and lifecycle methods. **Function components** are plain JavaScript functions that, since React 16.8's Hooks release, can do everything class components can — with significantly less code and complexity.

**Analogy**
Class components are like a formal letter written following strict rules: header, salutation, body, closing, signature, in that exact order. Function components are like texting: you just say what you need to say. Both communicate the same message — one just has far more required structure.

---

### 1. Side-by-Side — The Same Component Written Both Ways

**Theory**: The simplest way to understand the difference is to see the same UI written in both styles.

**Working Flow**
![flow-chart](flow-chart.png)

**Class component — Counter**
```jsx
import React, { Component } from "react";

class Counter extends Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
    this.handleIncrement = this.handleIncrement.bind(this); // manual binding
  }

  handleIncrement() {
    this.setState({ count: this.state.count + 1 });
  }

  componentDidMount() {
    document.title = `Count: ${this.state.count}`;
  }

  componentDidUpdate(prevProps, prevState) {
    if (prevState.count !== this.state.count) {
      document.title = `Count: ${this.state.count}`;
    }
  }

  componentWillUnmount() {
    document.title = "React App"; // cleanup
  }

  render() {
    return (
      <div>
        <h1>{this.props.label}: {this.state.count}</h1>
        <button onClick={this.handleIncrement}>Increment</button>
      </div>
    );
  }
}
```

**Function component — exact same behavior**
```jsx
import { useState, useEffect } from "react";

function Counter({ label }) {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `Count: ${count}`;
    return () => { document.title = "React App"; }; // cleanup on unmount
  }, [count]); // runs when count changes (replaces componentDidUpdate)

  return (
    <div>
      <h1>{label}: {count}</h1>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
    </div>
  );
}
```

**Output**
```
Both render identically:
  Count: 0  [Increment]
Click button → Count: 1  [Increment]
Browser tab title updates to "Count: 1" in both versions
```

**Line count comparison**
```
Class component:  ~30 lines
Function component: ~14 lines
Reduction: ~53% less code for the same behavior
```

---

### 2. State Management — this.setState vs useState

**Theory**: Class components store all state in a single `this.state` object and use `this.setState()` to update it. Function components use individual `useState` hooks for each piece of state. The function approach is more granular and easier to reason about.

**Working Flow**
![flow-chart-2](flow-chart-2.png)

**Class — multi-field form state**
```jsx
class LoginForm extends Component {
  constructor(props) {
    super(props);
    this.state = {
      email:    "",
      password: "",
      error:    null,
      loading:  false,
    };
  }

  handleSubmit(e) {
    e.preventDefault();
    this.setState({ loading: true, error: null });
    login(this.state.email, this.state.password)
      .then(() => this.setState({ loading: false }))
      .catch(err => this.setState({ loading: false, error: err.message }));
  }

  render() {
    const { email, password, error, loading } = this.state;
    return (
      <form onSubmit={this.handleSubmit.bind(this)}>
        <input value={email}    onChange={e => this.setState({ email: e.target.value })} />
        <input value={password} onChange={e => this.setState({ password: e.target.value })} />
        {error && <p>{error}</p>}
        <button disabled={loading}>
          {loading ? "Logging in..." : "Login"}
        </button>
      </form>
    );
  }
}
```

**Function — same form**
```jsx
function LoginForm() {
  const [email,    setEmail]    = useState("");
  const [password, setPassword] = useState("");
  const [error,    setError]    = useState(null);
  const [loading,  setLoading]  = useState(false);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setLoading(true);
    setError(null);
    try {
      await login(email, password);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input value={email}    onChange={e => setEmail(e.target.value)} />
      <input value={password} onChange={e => setPassword(e.target.value)} />
      {error && <p>{error}</p>}
      <button disabled={loading}>{loading ? "Logging in..." : "Login"}</button>
    </form>
  );
}
```

---

### 3. Lifecycle Methods vs useEffect

**Theory**: Class components have separate lifecycle methods for mounting, updating, and unmounting. `useEffect` consolidates all three into one hook, controlled by the dependency array.

**Working Flow**
![flow-chart-3](flow-chart-3.png)

**Class — lifecycle methods**
```jsx
class DataPanel extends Component {
  componentDidMount() {
    // Subscribe when component appears
    this.subscription = eventBus.subscribe(this.props.channel, this.handleEvent);
  }

  componentDidUpdate(prevProps) {
    // Re-subscribe if channel prop changes
    if (prevProps.channel !== this.props.channel) {
      eventBus.unsubscribe(this.subscription);
      this.subscription = eventBus.subscribe(this.props.channel, this.handleEvent);
    }
  }

  componentWillUnmount() {
    // Cleanup when component disappears
    eventBus.unsubscribe(this.subscription);
  }
}
```

**Function — useEffect consolidates all three**
```jsx
function DataPanel({ channel }) {
  useEffect(() => {
    const subscription = eventBus.subscribe(channel, handleEvent);
    return () => eventBus.unsubscribe(subscription); // cleanup
  }, [channel]); // re-runs when channel changes (handles componentDidUpdate too)
}
```

**Output**
```
Channel = "sports":    Subscribe to "sports"
Channel → "news":      Unsubscribe from "sports", Subscribe to "news"
Component unmounts:    Unsubscribe from "news"

Both class and function produce the same subscription behavior.
Function version is 6 lines vs ~15 lines for class.
```

---

### 4. The "this" Problem in Class Components

**Theory**: `this` in JavaScript refers to the execution context, which can change depending on how a function is called. In class components, forgetting to bind event handlers is one of the most common React bugs. Function components use closures instead of `this`, eliminating this entire category of errors.

**Working Flow**
![flow-chart-4](flow-chart-4.png)

**Class — the "this" bug**
```jsx
class ClickCounter extends Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
    // Option 1: bind in constructor (verbose)
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    // Without binding, "this" is undefined in strict mode
    this.setState({ count: this.state.count + 1 });
  }

  render() {
    return (
      <>
        {/* Option 2: bind inline (creates new function every render) */}
        <button onClick={this.handleClick.bind(this)}>Click</button>

        {/* Option 3: arrow function inline (also creates new function) */}
        <button onClick={() => this.handleClick()}>Click</button>
      </>
    );
  }
}
```

**Class — class field syntax (modern, avoids binding)**
```jsx
class ClickCounter extends Component {
  state = { count: 0 };

  // Arrow function class field — automatically bound to instance
  handleClick = () => {
    this.setState({ count: this.state.count + 1 });
  };

  render() {
    return <button onClick={this.handleClick}>Click</button>;
  }
}
```

**Function — no "this" problem**
```jsx
function ClickCounter() {
  const [count, setCount] = useState(0);

  // No binding needed — just a regular function in scope
  const handleClick = () => setCount(c => c + 1);

  return <button onClick={handleClick}>Click</button>;
}
```

---

### 5. Logic Sharing — Hooks vs HOC and Render Props

**Theory**: One of the biggest limitations of class components is that stateful logic cannot be reused across components without patterns like HOC (Higher-Order Components) or render props — both of which add wrapper layers and complexity. Function components share logic through custom hooks — a far cleaner model.

**Working Flow**
![flow-chart-5](flow-chart-5.png)

**Class — sharing data fetch logic (HOC pattern)**
```jsx
// Reusable HOC — lots of boilerplate
function withFetch(url, WrappedComponent) {
  return class extends Component {
    state = { data: null, loading: true };
    componentDidMount() {
      fetch(url).then(r => r.json()).then(data => this.setState({ data, loading: false }));
    }
    render() {
      return <WrappedComponent {...this.props} data={this.state.data} loading={this.state.loading} />;
    }
  };
}

const UserListWithFetch = withFetch("/api/users", UserList);
```

**Function — sharing data fetch logic (custom hook)**
```jsx
// Custom hook — clean, no wrapper
function useFetch(url) {
  const [data, setData]       = useState(null);
  const [loading, setLoading] = useState(true);
  useEffect(() => {
    fetch(url).then(r => r.json()).then(d => { setData(d); setLoading(false); });
  }, [url]);
  return { data, loading };
}

function UserList() {
  const { data, loading } = useFetch("/api/users");
  if (loading) return <p>Loading...</p>;
  return <ul>{data.map(u => <li key={u.id}>{u.name}</li>)}</ul>;
}

function ProductList() {
  const { data, loading } = useFetch("/api/products");  // reuse same hook
  if (loading) return <p>Loading...</p>;
  return <ul>{data.map(p => <li key={p.id}>{p.name}</li>)}</ul>;
}
```

---

### 6. When Would You Still Use a Class Component?

**Theory**: For new code, always use function components. Class components remain relevant in two specific scenarios: maintaining legacy codebases, and implementing Error Boundaries (which still require class syntax as of React 18).

**Working Flow**
![flow-chart-6](flow-chart-6.png)

**Error Boundary — class still required**
```jsx
// This MUST be a class — no functional equivalent in core React
class RouteErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError() { return { hasError: true }; }
  componentDidCatch(error, info)    { logToSentry(error, info); }

  render() {
    if (this.state.hasError) return <PageCrashFallback />;
    return this.props.children;
  }
}
```

**Comparison summary**
```text
Feature                  Class Component         Function Component
─────────────────────    ──────────────────────  ──────────────────────
Syntax                   ES6 class               Plain function
State                    this.state + setState    useState hook
Lifecycle                Separate methods        useEffect hook
Logic sharing            HOC / render props       Custom hooks
"this" binding           Required                Not needed
Boilerplate              High                    Low
Performance              Similar                 Slightly easier to optimize
Error Boundaries         Yes (required)          No (class still needed)
New code recommendation  No — legacy only        Yes — always
```

---

### Real-World Migration Example

```jsx
// BEFORE — class component (legacy code)
class UserGreeting extends Component {
  constructor(props) {
    super(props);
    this.state = { time: new Date().toLocaleTimeString() };
  }
  componentDidMount() {
    this.timer = setInterval(() => {
      this.setState({ time: new Date().toLocaleTimeString() });
    }, 1000);
  }
  componentWillUnmount() {
    clearInterval(this.timer);
  }
  render() {
    return <h1>Hello, {this.props.name}! It is {this.state.time}.</h1>;
  }
}

// AFTER — function component (modern equivalent)
function UserGreeting({ name }) {
  const [time, setTime] = useState(new Date().toLocaleTimeString());

  useEffect(() => {
    const timer = setInterval(() => {
      setTime(new Date().toLocaleTimeString());
    }, 1000);
    return () => clearInterval(timer);
  }, []);

  return <h1>Hello, {name}! It is {time}.</h1>;
}
```

**Output**
```
Hello, Alice! It is 14:32:05.
(updates every second)
Hello, Alice! It is 14:32:06.
Hello, Alice! It is 14:32:07.
```

---

[View Interview Questions](./interview.md)
