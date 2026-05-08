# Class vs Function Components — Interview Questions

---

**1. What is the main syntactic difference between class and function components?**

A class component extends `React.Component`, requires a `render()` method, and uses `this` to access props and state. A function component is a plain JavaScript function that receives props as a parameter and returns JSX directly.

```jsx
// Class component
class Greeting extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}

// Function component — same output, much less code
function Greeting({ name }) {
  return <h1>Hello, {name}</h1>;
}
```

---

**2. How is state managed differently in class vs function components?**

Class components use `this.state` (an object) and `this.setState()` which merges updates. Function components use individual `useState` hooks, each managing one piece of state independently.

```jsx
// Class — all state in one object, setState merges
class Form extends React.Component {
  state = { email: "", password: "", error: null };

  updateEmail = (e) => this.setState({ email: e.target.value }); // merges, keeps password
  updatePassword = (e) => this.setState({ password: e.target.value });

  render() {
    return (
      <>
        <input value={this.state.email}    onChange={this.updateEmail} />
        <input value={this.state.password} onChange={this.updatePassword} />
      </>
    );
  }
}

// Function — separate state for each value
function Form() {
  const [email,    setEmail]    = useState("");
  const [password, setPassword] = useState("");
  const [error,    setError]    = useState(null);

  return (
    <>
      <input value={email}    onChange={e => setEmail(e.target.value)} />
      <input value={password} onChange={e => setPassword(e.target.value)} />
    </>
  );
}
```

---

**3. How do you replicate lifecycle methods in function components?**

`useEffect` consolidates `componentDidMount`, `componentDidUpdate`, and `componentWillUnmount` into one hook, controlled by the dependency array.

```jsx
// Class — three separate lifecycle methods
class Timer extends React.Component {
  componentDidMount()              { this.id = setInterval(this.tick, 1000); }
  componentWillUnmount()           { clearInterval(this.id); }
  componentDidUpdate(prevProps)    {
    if (prevProps.interval !== this.props.interval) {
      clearInterval(this.id);
      this.id = setInterval(this.tick, this.props.interval);
    }
  }
}

// Function — one useEffect handles all three cases
function Timer({ interval }) {
  useEffect(() => {
    const id = setInterval(tick, interval);
    return () => clearInterval(id); // cleanup = componentWillUnmount
  }, [interval]); // re-run when interval changes = componentDidUpdate
}
```

---

**4. What is the "this binding" problem in class components and why don't function components have it?**

In JavaScript, `this` inside a method refers to whatever called the function. When React calls an event handler, `this` is not the component instance — it's `undefined` (in strict mode). Function components use closures instead of `this`, so they never have this issue.

```jsx
// Class — binding is required
class Button extends React.Component {
  constructor(props) {
    super(props);
    this.handleClick = this.handleClick.bind(this); // required!
  }

  handleClick() {
    console.log(this.props.label); // "this" is the component
  }

  render() {
    return <button onClick={this.handleClick}>{this.props.label}</button>;
  }
}

// Without bind — crashes: "Cannot read properties of undefined (reading 'props')"
// Because onClick calls handleClick without "this" context

// Function — no binding, no problem
function Button({ label }) {
  const handleClick = () => {
    console.log(label); // "label" is just a variable in scope — no "this"
  };
  return <button onClick={handleClick}>{label}</button>;
}
```

---

**5. Is there any situation where you must use a class component today?**

Yes: Error Boundaries. `getDerivedStateFromError` and `componentDidCatch` have no hook equivalents in core React. For all other use cases, function components are preferred.

```jsx
// Must be a class — no functional alternative
class ErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError(error) {
    return { hasError: true };  // no hook equivalent
  }

  componentDidCatch(error, info) {
    logToSentry(error, info);   // no hook equivalent
  }

  render() {
    if (this.state.hasError) return <h2>Something went wrong.</h2>;
    return this.props.children;
  }
}

// For everything else — function components are preferred:
// State       → useState / useReducer
// Lifecycle   → useEffect
// Refs        → useRef
// Context     → useContext
// Performance → useMemo / useCallback
```

---

**6. How do you share stateful logic between class components vs function components?**

Class components use Higher-Order Components (HOC) or render props patterns — both add wrapper layers. Function components use custom hooks — much simpler, no wrapper needed.

```jsx
// Class — HOC pattern (complex, adds nesting)
function withWindowSize(Component) {
  return class extends React.Component {
    state = { width: window.innerWidth };
    componentDidMount() {
      window.addEventListener("resize", () => this.setState({ width: window.innerWidth }));
    }
    render() {
      return <Component {...this.props} windowWidth={this.state.width} />;
    }
  };
}
const ResponsiveNavbar = withWindowSize(Navbar);

// Function — custom hook (simple, no wrapper)
function useWindowSize() {
  const [width, setWidth] = useState(window.innerWidth);
  useEffect(() => {
    const handler = () => setWidth(window.innerWidth);
    window.addEventListener("resize", handler);
    return () => window.removeEventListener("resize", handler);
  }, []);
  return width;
}

function Navbar() {
  const width = useWindowSize(); // zero wrappers
  return <nav>{width < 768 ? <MobileMenu /> : <DesktopMenu />}</nav>;
}
```

---

**7. Which is more performant — class or function components?**

In practice, performance is comparable. Function components avoid the overhead of creating a class instance and are more easily tree-shaken and minified. More importantly, React's optimization tools (memo, useCallback, useMemo) are designed around function components. React's future optimizations (React Compiler) also target function components.

```jsx
// Function component with memoization — fine-grained optimization
const ProductCard = React.memo(function ProductCard({ product, onAdd }) {
  return (
    <div>
      <h3>{product.name}</h3>
      <button onClick={() => onAdd(product.id)}>Add</button>
    </div>
  );
});

// Stable callback reference — prevents unnecessary re-renders of ProductCard
function ProductList({ products }) {
  const handleAdd = useCallback((id) => {
    addToCart(id);
  }, []); // stable reference

  return products.map(p => <ProductCard key={p.id} product={p} onAdd={handleAdd} />);
}
```

---

**8. What is the equivalent of the constructor in a function component?**

There is no constructor equivalent. State initialization goes directly in `useState`, side effects go in `useEffect`, and one-time computed values go in `useState`'s lazy initializer or `useMemo`.

```jsx
// Class — constructor initializes everything
class UserPage extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      user: null,
      processedId: processId(props.userId), // computed once
    };
  }
}

// Function — no constructor needed
function UserPage({ userId }) {
  const [user, setUser] = useState(null);

  // Lazy initializer — runs once on mount (expensive computation)
  const processedId = useMemo(() => processId(userId), [userId]);

  // OR: lazy useState initializer (runs once, never again)
  const [config] = useState(() => buildConfig(userId));
}
```

---

**9. How does componentDidUpdate compare to useEffect with dependencies?**

`componentDidUpdate` runs after every render and receives previous props/state for comparison. `useEffect` with a dependency array is more declarative — you list what to watch, and React handles the comparison automatically.

```jsx
// Class — manual comparison with prevProps
componentDidUpdate(prevProps) {
  if (prevProps.userId !== this.props.userId) {
    this.fetchUser(this.props.userId);
  }
  if (prevProps.theme !== this.props.theme) {
    this.updateTheme(this.props.theme);
  }
}

// Function — separate effects for separate concerns (cleaner)
useEffect(() => {
  fetchUser(userId);
}, [userId]); // runs when userId changes

useEffect(() => {
  updateTheme(theme);
}, [theme]); // runs when theme changes

// Each effect has one responsibility — no mixing of concerns
```

---

**10. Should you rewrite existing class components to function components?**

Not necessarily — React fully supports class components with no deprecation plans. Rewrites carry risk and cost development time. The practical rule: keep existing class components as-is, write all new components as function components.

```text
Decision guide:
  New component needed?           → Always write as function component
  Existing class works fine?      → Leave it — no urgency to rewrite
  Existing class is complex?      → Consider gradual migration only if actively maintained
  Adding new feature to old class?→ You can add the feature as a function component
                                    that the class renders as a child

When to migrate a class component:
  ✓ The component is a major pain point to maintain
  ✓ You need to extract shared logic (easier with hooks)
  ✓ The team has capacity and the area has good test coverage

When NOT to migrate:
  ✗ "It works" and isn't being actively changed
  ✗ No test coverage (migration risk is high)
  ✗ Just to follow a trend (technical debt without ROI)
```

```jsx
// Pragmatic approach — old class component stays, new child is functional
class LegacyDashboard extends React.Component {
  state = { data: null };
  componentDidMount() { fetchDashboardData().then(d => this.setState({ data: d })); }

  render() {
    return (
      <div>
        <LegacyHeader />
        {/* New feature added as a function component — no class rewrite needed */}
        <NewAnalyticsPanel data={this.state.data} />
      </div>
    );
  }
}

function NewAnalyticsPanel({ data }) {
  const [view, setView] = useState("chart");
  return (/* modern JSX */);
}
```
