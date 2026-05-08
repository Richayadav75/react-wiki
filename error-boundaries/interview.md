# Error Boundaries — Interview Questions

---

**1. What is an Error Boundary and what problem does it solve?**

An Error Boundary is a class component that wraps a subtree and catches any JavaScript error thrown during rendering in that subtree. Without it, any render error causes React to unmount the entire app (blank white page). Error Boundaries let the rest of the app keep working while showing a fallback UI for the broken section.

```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) return <p>Something went wrong.</p>;
    return this.props.children;
  }
}

// App keeps working even if ProductCard throws
function App() {
  return (
    <div>
      <Navbar />                               {/* unaffected by card error */}
      <ErrorBoundary fallback={<p>Card error</p>}>
        <ProductCard product={brokenData} />   {/* may throw */}
      </ErrorBoundary>
      <Footer />                               {/* unaffected */}
    </div>
  );
}
```

---

**2. Why must Error Boundaries be class components?**

Because they rely on two lifecycle methods — `getDerivedStateFromError` and `componentDidCatch` — that have no functional hook equivalents in core React. React's error handling hooks into the class component lifecycle specifically. React 19 (and the `react-error-boundary` library) offers functional alternatives, but the underlying boundary must still be a class.

```jsx
class ErrorBoundary extends React.Component {
  // getDerivedStateFromError — called in the render phase, must be static
  static getDerivedStateFromError(error) {
    return { hasError: true }; // triggers re-render with fallback
  }

  // componentDidCatch — called in the commit phase, safe for side effects
  componentDidCatch(error, info) {
    logErrorToSentry(error, info.componentStack); // logging, analytics
  }

  render() {
    if (this.state.hasError) return this.props.fallback;
    return this.props.children;
  }
}
```

---

**3. What is the difference between getDerivedStateFromError and componentDidCatch?**

| Method | Phase | Purpose | Can cause side effects? |
|---|---|---|---|
| `getDerivedStateFromError` | Render phase | Update state to show fallback | No — must be pure |
| `componentDidCatch` | Commit phase | Log the error to a service | Yes — safe for side effects |

```jsx
static getDerivedStateFromError(error) {
  // Pure — just return the state update
  return { hasError: true, errorMessage: error.message };
}

componentDidCatch(error, errorInfo) {
  // Side effects OK here
  fetch("/api/errors", {
    method: "POST",
    body: JSON.stringify({ error: error.message, stack: errorInfo.componentStack }),
  });
}
```

---

**4. What errors does an Error Boundary NOT catch?**

```text
NOT caught by Error Boundary:
  ✗ Event handler errors (onClick, onChange, onSubmit)
  ✗ Asynchronous code (setTimeout, fetch, async/await in useEffect)
  ✗ Server-side rendering errors
  ✗ Errors thrown by the Error Boundary itself
```

```jsx
// NOT caught — error in event handler
function Button() {
  const handleClick = () => {
    null.toString(); // TypeError — ErrorBoundary does NOT catch this
  };
  return <button onClick={handleClick}>Click</button>;
}

// FIX — use try/catch in the handler
function Button() {
  const handleClick = () => {
    try {
      null.toString();
    } catch (err) {
      setError(err.message); // handle locally
    }
  };
  return <button onClick={handleClick}>Click</button>;
}

// NOT caught — async error in useEffect
function Component() {
  useEffect(() => {
    fetch("/api").then(r => r.json()).then(d => {
      if (!d) throw new Error("empty"); // NOT caught by boundary
    });
  }, []);
}

// FIX — re-throw in render phase via state
function Component() {
  const [error, setError] = useState(null);
  if (error) throw error; // ErrorBoundary catches this

  useEffect(() => {
    fetch("/api").catch(err => setError(err));
  }, []);
}
```

---

**5. Where is the best place to put Error Boundaries in an application?**

At multiple granularity levels: one at the root (last resort), one per route/page (isolates page crashes), and one per high-risk widget (isolates individual features).

```jsx
// 1. Root boundary — last resort for anything that slips through
function App() {
  return (
    <ErrorBoundary fallback={<AppCrashPage />}>
      <Router />
    </ErrorBoundary>
  );
}

// 2. Route-level boundary — each page is isolated
<Route path="/dashboard" element={
  <ErrorBoundary fallback={<PageError page="Dashboard" />}>
    <DashboardPage />
  </ErrorBoundary>
} />

// 3. Widget-level boundary — high-risk UI components
function DashboardPage() {
  return (
    <div>
      <ErrorBoundary fallback={<p>Chart unavailable</p>}>
        <RevenueChart />       {/* third-party, may throw with bad data */}
      </ErrorBoundary>
      <WelcomeMessage />       {/* stable — no boundary needed */}
    </div>
  );
}
```

---

**6. How do you reset an Error Boundary so the user can retry?**

Add a "Try again" button that calls `setState({ hasError: false })`. For automatic reset when the resource changes (e.g., user navigates to a new product), check `prevProps` in `componentDidUpdate`.

```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError() { return { hasError: true }; }

  componentDidUpdate(prevProps) {
    // Reset when a key resource changes
    if (this.state.hasError && prevProps.resourceKey !== this.props.resourceKey) {
      this.setState({ hasError: false });
    }
  }

  render() {
    if (this.state.hasError) {
      return (
        <div>
          <p>Failed to load.</p>
          <button onClick={() => this.setState({ hasError: false })}>
            Try again
          </button>
        </div>
      );
    }
    return this.props.children;
  }
}

// Auto-reset when productId changes
function ProductPage({ productId }) {
  return (
    <ErrorBoundary resourceKey={productId}>
      <ProductDetail productId={productId} />
    </ErrorBoundary>
  );
}
```

---

**7. What is the react-error-boundary library and why use it?**

`react-error-boundary` is a popular library that provides a pre-built, flexible `ErrorBoundary` component and the `useErrorBoundary` hook for functional components. It reduces boilerplate compared to writing class boundaries from scratch.

```jsx
import { ErrorBoundary, useErrorBoundary } from "react-error-boundary";

function Fallback({ error, resetErrorBoundary }) {
  return (
    <div>
      <p>Error: {error.message}</p>
      <button onClick={resetErrorBoundary}>Retry</button>
    </div>
  );
}

// Functional component can trigger boundary for async errors
function DataWidget({ userId }) {
  const { showBoundary } = useErrorBoundary();

  useEffect(() => {
    fetchUser(userId).catch(err => showBoundary(err)); // async → boundary
  }, [userId]);

  return <p>User data loaded</p>;
}

function App() {
  return (
    <ErrorBoundary
      FallbackComponent={Fallback}
      onError={(err, info) => logToSentry(err, info)}
    >
      <DataWidget userId={1} />
    </ErrorBoundary>
  );
}
```

---

**8. How do you log errors from an Error Boundary to Sentry (or similar)?**

Use `componentDidCatch` — it receives the error object and `errorInfo` with the component stack. Call the logging service there.

```jsx
componentDidCatch(error, errorInfo) {
  // Sentry
  Sentry.withScope(scope => {
    scope.setExtras(errorInfo);
    Sentry.captureException(error);
  });

  // Or any custom endpoint
  fetch("/api/log-error", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      message:    error.message,
      stack:      error.stack,
      component:  errorInfo.componentStack,
      timestamp:  new Date().toISOString(),
    }),
  });
}
```

---

**9. What happens if an error is thrown inside the Error Boundary's own render method?**

The error propagates upward to the next Error Boundary in the tree. An Error Boundary cannot catch its own errors — only those in its children. This is why you should always have a root-level boundary as a safety net.

```jsx
class BuggyBoundary extends React.Component {
  render() {
    if (this.state.hasError) {
      null.toString(); // throws in the boundary itself!
    }
    return this.props.children;
  }
}

// This error propagates to the PARENT ErrorBoundary (or crashes the app if none)
// Always nest ErrorBoundaries or keep the render() of an ErrorBoundary very simple
```

---

**10. What is the difference between Error Boundaries and try/catch?**

```text
try/catch:
  → Works in JavaScript: synchronous code, event handlers, async/await
  → Cannot catch errors during React's render phase
  → Stops at the function boundary — does not interact with React's component tree

Error Boundary:
  → Works for React's render phase and lifecycle methods
  → Interacts with React's component tree — can show fallback UI
  → Cannot catch errors in event handlers or async code

Use try/catch:   inside event handlers, inside async functions, inside utility code
Use ErrorBoundary: around component subtrees that might throw during rendering
```

```jsx
// try/catch — correct for event handler
async function handleSubmit() {
  try {
    await submitOrder(cart);
    setSuccess(true);
  } catch (err) {
    setSubmitError(err.message); // update state — no ErrorBoundary needed
  }
}

// ErrorBoundary — correct for render-time failure
function OrderSummary({ orderId }) {
  const order = getOrderOrThrow(orderId); // might throw during render
  return <div>{order.items.map(i => <li>{i.name}</li>)}</div>;
}

// Wrap OrderSummary in ErrorBoundary:
<ErrorBoundary fallback={<p>Order details unavailable</p>}>
  <OrderSummary orderId={id} />
</ErrorBoundary>
```
