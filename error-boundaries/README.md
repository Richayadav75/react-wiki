- Category: React Patterns
- Difficulty: Intermediate
- Related: error-handling, component-lifecycle, class-vs-function-component

### Error Boundaries — Graceful Failure in React UIs
An **Error Boundary** is a React component that catches JavaScript errors anywhere in its child component tree, prevents the entire app from crashing, and displays a fallback UI in place of the broken component tree. Without error boundaries, a single runtime error in a deeply nested component would unmount your entire application.

**Analogy**
Think of circuit breakers in a building's electrical system. If one room's circuit overloads (an error occurs), only that circuit trips — the rest of the building keeps its power. Error boundaries are React's circuit breakers: a crash in the Sidebar doesn't take down the Navbar, the main content, or any other part of the app.

---

### 1. Why Error Boundaries Exist

**Theory**: In React 16+, an uncaught error during rendering causes React to unmount the entire component tree. This is intentional — React would rather show nothing than display a corrupt UI. But "nothing" is still a terrible user experience. Error Boundaries let you choose what to show instead.

**Working Flow**
![flow-chart](flow-chart.png)

**Example — without vs with**
```jsx
// Without error boundary — one broken ProductCard kills the whole app
function ProductGrid({ products }) {
  return (
    <div>
      {products.map(p => <ProductCard key={p.id} product={p} />)}
    </div>
  );
}

// With error boundary — broken card shows fallback, others stay
function ProductGrid({ products }) {
  return (
    <div>
      {products.map(p => (
        <ErrorBoundary key={p.id} fallback={<p>Failed to load product</p>}>
          <ProductCard product={p} />
        </ErrorBoundary>
      ))}
    </div>
  );
}
```

**Output**
```
Without boundary:   [blank white page]

With boundary:
  [Product 1 card]
  Failed to load product   ← graceful fallback for the broken card
  [Product 3 card]
  [Product 4 card]
```

---

### 2. How to Implement an Error Boundary

**Theory**: Error boundaries must be **class components**. There are two lifecycle methods that turn a class into an error boundary:
- `static getDerivedStateFromError(error)` — updates state to trigger fallback render
- `componentDidCatch(error, info)` — side effect for logging (Sentry, etc.)

As of React 19, a `useErrorBoundary` hook is available in some libraries, but the class component approach remains the standard for custom boundaries.

**Working Flow**
![flow-chart-2](flow-chart-2.png)

**Basic implementation**
```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  // Called during the render phase — must be static, must return state update
  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  // Called after render — safe place for side effects like logging
  componentDidCatch(error, errorInfo) {
    console.error("Error caught by boundary:", error);
    console.error("Component stack:", errorInfo.componentStack);

    // Send to error tracking service
    // logToSentry(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // Render the fallback UI — can use this.props.fallback for flexibility
      return this.props.fallback || (
        <div className="error-fallback">
          <h2>Something went wrong.</h2>
          <p>{this.state.error?.message}</p>
          <button onClick={() => this.setState({ hasError: false, error: null })}>
            Try again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}
```

**Flexible, reusable error boundary**
```jsx
// Accepts a fallback prop for custom UI per use case
<ErrorBoundary fallback={<p>Chart failed to load</p>}>
  <RevenueChart data={data} />
</ErrorBoundary>

<ErrorBoundary fallback={<UserProfileSkeleton />}>
  <UserProfile userId={userId} />
</ErrorBoundary>
```

---

### 3. What Error Boundaries DO and DON'T Catch

**Theory**: Error Boundaries only catch errors that occur in the render phase and lifecycle methods of their children. They cannot catch errors in async code, event handlers, or the boundary itself.

**Working Flow**
![flow-chart-3](flow-chart-3.png)

**Example — caught vs not caught**
```jsx
// CAUGHT — throws during render
function BadComponent({ data }) {
  const result = data.items.map(i => i.name);  // TypeError if data is null
  return <ul>{result.map(n => <li>{n}</li>)}</ul>;
}
// → ErrorBoundary catches this

// NOT CAUGHT — error in event handler
function BadButton() {
  const handleClick = () => {
    null.toString(); // TypeError in event handler
  };
  return <button onClick={handleClick}>Click</button>;
}
// → ErrorBoundary does NOT catch this — use try/catch inside the handler

// FIX for event handler errors:
function SafeButton() {
  const handleClick = () => {
    try {
      null.toString();
    } catch (error) {
      console.error("Button error:", error);
      // show toast notification, update local state, etc.
    }
  };
  return <button onClick={handleClick}>Click</button>;
}

// NOT CAUGHT — async error
function AsyncComponent() {
  useEffect(() => {
    fetch("/api/data")
      .then(r => r.json())
      .then(d => {
        if (!d) throw new Error("No data"); // NOT caught by ErrorBoundary
      });
  }, []);
}

// FIX — manually move async errors into React's render phase
function AsyncComponent() {
  const [error, setError] = useState(null);
  if (error) throw error;  // re-throw in render → ErrorBoundary catches it

  useEffect(() => {
    fetch("/api/data")
      .then(r => r.json())
      .catch(err => setError(err));  // store error in state → triggers re-render → throw
  }, []);
}
```

---

### 4. Where to Place Error Boundaries

**Theory**: Placement is a strategic decision. Too few boundaries (just one at the root) means any error kills the entire UI. Too many boundaries adds unnecessary complexity. Wrap sections of your app that are independently meaningful and can degrade gracefully.

**Working Flow**
![flow-chart-4](flow-chart-4.png)

**Example — route-level boundary**
```jsx
function App() {
  return (
    <BrowserRouter>
      <Navbar />
      <Routes>
        <Route
          path="/dashboard"
          element={
            <ErrorBoundary fallback={<PageError message="Dashboard failed to load" />}>
              <DashboardPage />
            </ErrorBoundary>
          }
        />
        <Route
          path="/profile"
          element={
            <ErrorBoundary fallback={<PageError message="Profile unavailable" />}>
              <ProfilePage />
            </ErrorBoundary>
          }
        />
      </Routes>
      <Footer />
    </BrowserRouter>
  );
}
// If DashboardPage crashes, Navbar and Footer still work. ProfilePage is unaffected.
```

**Example — widget-level boundary**
```jsx
function DashboardPage() {
  return (
    <div className="dashboard">
      <h1>Dashboard</h1>

      {/* Wrap high-risk third-party or data-heavy widgets */}
      <ErrorBoundary fallback={<p>Chart unavailable</p>}>
        <RevenueChart />
      </ErrorBoundary>

      <ErrorBoundary fallback={<p>Feed temporarily unavailable</p>}>
        <ActivityFeed />
      </ErrorBoundary>

      {/* Low-risk component — no boundary needed */}
      <WelcomeMessage />
    </div>
  );
}
```

---

### 5. Resetting Error Boundaries

**Theory**: Once an error is caught, the boundary stays in the error state until something resets it. You can provide a "Try Again" button that resets internal state, or use a `resetKeys` pattern to automatically reset when certain props change.

**Working Flow**
![flow-chart-5](flow-chart-5.png)

**Example — try again button**
```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError() { return { hasError: true }; }
  componentDidCatch(error, info)    { logError(error, info); }

  // Auto-reset when key props change (e.g., when user navigates to a new resource)
  componentDidUpdate(prevProps) {
    if (this.state.hasError && prevProps.resetKey !== this.props.resetKey) {
      this.setState({ hasError: false });
    }
  }

  render() {
    if (this.state.hasError) {
      return (
        <div>
          <p>This section failed to load.</p>
          <button onClick={() => this.setState({ hasError: false })}>
            Try again
          </button>
        </div>
      );
    }
    return this.props.children;
  }
}

// Usage — resetKey changes on every navigation → boundary resets automatically
function ProductPage({ productId }) {
  return (
    <ErrorBoundary resetKey={productId}>
      <ProductDetail productId={productId} />
    </ErrorBoundary>
  );
}
```

---

### 6. React 19 — useErrorBoundary Hook (New)

**Theory**: React 19 introduces `useErrorBoundary` (also available earlier via the `react-error-boundary` library). This allows functional components to trigger and reset error boundaries, bringing the pattern closer to the hooks model.

**Working Flow**
![flow-chart-6](flow-chart-6.png)

**Example — using react-error-boundary library**
```jsx
import { ErrorBoundary, useErrorBoundary } from "react-error-boundary";

function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div role="alert">
      <p>Something went wrong:</p>
      <pre>{error.message}</pre>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}

function UserProfile({ userId }) {
  const { showBoundary } = useErrorBoundary();

  useEffect(() => {
    fetchUser(userId).catch(showBoundary); // async error → triggers boundary
  }, [userId]);

  return <p>User profile content</p>;
}

function App() {
  return (
    <ErrorBoundary
      FallbackComponent={ErrorFallback}
      onError={(error, info) => logToSentry(error, info)}
      onReset={() => console.log("Boundary reset")}
    >
      <UserProfile userId={1} />
    </ErrorBoundary>
  );
}
```

---

### Real-World Examples

```text
SaaS Dashboard:
  Route-level boundary → wraps each page route
  Widget-level boundary → wraps each dashboard widget
  Both: show skeleton/retry fallback, log to Sentry

E-commerce:
  Product card boundary → broken product shows "Unavailable" card, others stay
  Checkout form boundary → if form crashes, show "Please refresh" + support contact

Third-party integrations:
  Wrap Google Maps, Stripe Elements, Intercom widgets
  If the third-party script throws, the rest of the page is unaffected

Reporting / Charts:
  Charts often throw with bad data (division by zero, null values)
  Wrap each chart in a boundary → chart shows "Data unavailable" while table shows fine
```

---

[View Interview Questions](./interview.md)
