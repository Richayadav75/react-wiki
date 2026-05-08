- Category: React Core
- Difficulty: Intermediate
- Related: hooks, props-drilling, useContext, redux, state

### Context API — sharing data without prop drilling

The Context API is React's built-in solution for passing data deeply through a component tree without manually threading it as props at every level. It is designed for *global* data that many components at different nesting levels need to access.

**Analogy**
A radio broadcast: the station (Provider) transmits a signal (context value). Any radio (component) in range can tune in (useContext) and receive the signal directly — without the signal passing through every building between the station and the listener.

---

### 1. The Three Steps of Context

**Theory**
Using Context always involves three steps: create it, provide it, consume it. Each step is a distinct operation.

**Working Flow**
![flow-chart](flow-chart.png)

**Example**
```jsx
import { createContext, useContext } from "react";

// Step 1 — create
const ThemeContext = createContext("light"); // "light" is the default fallback

// Step 2 — provide
function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Page />
    </ThemeContext.Provider>
  );
}

// Step 3 — consume (anywhere below the Provider)
function Page() {
  return <Button />;               // no theme prop needed here
}

function Button() {
  const theme = useContext(ThemeContext); // gets "dark" directly
  return (
    <button style={{ background: theme === "dark" ? "#333" : "#fff" }}>
      Current theme: {theme}
    </button>
  );
}
```

**Output**
```
Rendered: button with dark background, text "Current theme: dark"
```

**Explanation**
`createContext` takes a default value — this is only used when a component calls `useContext` but has NO Provider anywhere above it in the tree. In practice you almost always have a Provider, so the default is a fallback safety net.

---

### 2. Dynamic Context (Context + State)

**Theory**
Static values in context are rarely enough. In real apps you combine context with `useState` so the context value can change over time (e.g., toggling dark/light mode). Pass both the value and the setter function inside the Provider's `value`.

**Working Flow**
![flow-chart-2](flow-chart-2.png)

**Example**
```jsx
import { createContext, useContext, useState } from "react";

const ThemeContext = createContext({ theme: "light", setTheme: () => {} });

function App() {
  const [theme, setTheme] = useState("light");

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <Navbar />
      <main>
        <ContentArea />
      </main>
    </ThemeContext.Provider>
  );
}

function Navbar() {
  const { theme, setTheme } = useContext(ThemeContext);

  return (
    <nav style={{ background: theme === "dark" ? "#1a1a1a" : "#f0f0f0" }}>
      <span>My App</span>
      <button onClick={() => setTheme(t => t === "light" ? "dark" : "light")}>
        Switch to {theme === "light" ? "Dark" : "Light"} Mode
      </button>
    </nav>
  );
}

function ContentArea() {
  const { theme } = useContext(ThemeContext);

  return (
    <div style={{
      background: theme === "dark" ? "#222" : "#fff",
      color: theme === "dark" ? "#eee" : "#111",
      padding: "24px"
    }}>
      <h1>Hello!</h1>
      <p>Theme is: {theme}</p>
    </div>
  );
}
```

**Output**
```
Light mode:
  Navbar: light gray background — [Switch to Dark Mode]
  Content: white background, dark text

Click toggle:
  Navbar: dark background — [Switch to Light Mode]
  Content: dark background, light text
```

---

### 3. Multiple Contexts

**Theory**
An app typically has several independent global concerns: theme, auth, locale, notifications. Rather than cramming all of them into one context (causing every consumer to re-render on any change), split them into separate focused contexts. Nest the Providers.

**Working Flow**
![flow-chart-3](flow-chart-3.png)

**Example**
```jsx
import { createContext, useContext, useState } from "react";

// Two separate contexts
const AuthContext  = createContext(null);
const ThemeContext = createContext("light");

function App() {
  const [user, setUser]   = useState(null);
  const [theme, setTheme] = useState("light");

  function login()  { setUser({ name: "Alice", role: "admin" }); }
  function logout() { setUser(null); }

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      <ThemeContext.Provider value={{ theme, setTheme }}>
        <Header />
        <Dashboard />
      </ThemeContext.Provider>
    </AuthContext.Provider>
  );
}

function Header() {
  const { user, logout }  = useContext(AuthContext);   // reads auth
  const { theme, setTheme } = useContext(ThemeContext); // reads theme

  return (
    <header>
      <span>{user ? `Logged in as ${user.name}` : "Guest"}</span>
      {user && <button onClick={logout}>Log Out</button>}
      <button onClick={() => setTheme(t => t === "light" ? "dark" : "light")}>
        Theme: {theme}
      </button>
    </header>
  );
}

function Dashboard() {
  const { user }  = useContext(AuthContext);
  const { theme } = useContext(ThemeContext);

  if (!user) return <p>Please log in.</p>;
  return (
    <div style={{ background: theme === "dark" ? "#222" : "#fff" }}>
      Welcome, {user.name}! You are a {user.role}.
    </div>
  );
}
```

**Output**
```
Guest + Light:  "Guest"         [Theme: light]   "Please log in."
After login:    "Logged in as Alice" [Log Out] [Theme: light]
                "Welcome, Alice! You are a admin."
Toggle theme:   same but with dark background
```

---

### 4. Context Re-render Behaviour

**Theory**
Every component that calls `useContext(SomeContext)` will re-render whenever the context *value* changes. If your Provider wraps the value in a new object literal on every render (`value={{ theme, setTheme }}`), the object reference changes on every render — causing all consumers to re-render even if theme and setTheme did not change.

**Working Flow**
![flow-chart-4](flow-chart-4.png)

**Example**
```jsx
import { createContext, useContext, useState, useMemo } from "react";

const ThemeContext = createContext({ theme: "light", setTheme: () => {} });

function App() {
  const [theme, setTheme] = useState("light");

  // Memoize the context value to avoid unnecessary re-renders
  const contextValue = useMemo(() => ({ theme, setTheme }), [theme]);

  return (
    <ThemeContext.Provider value={contextValue}>
      <Consumer />
    </ThemeContext.Provider>
  );
}
```

**Output**
```
WITHOUT useMemo: every re-render of App → Consumer re-renders
WITH useMemo:    Consumer only re-renders when theme actually changes
```

---

### 5. Context vs Redux — When to Use Each

**Theory**
Context API is a *data distribution mechanism*, not a state management solution. It does not provide middleware, devtools, time-travel debugging, or optimised subscriptions. Redux is a complete state management library. The choice depends on complexity.

**Working Flow**
![flow-chart-5](flow-chart-5.png)

**Comparison Table**

| Feature               | Context API           | Redux                          |
|-----------------------|-----------------------|--------------------------------|
| Setup                 | Built-in, minimal     | npm install, boilerplate       |
| State location        | React tree (Provider) | External store                 |
| Devtools              | No                    | Yes (Redux DevTools)           |
| Middleware / async    | Manual (useEffect)    | redux-thunk, redux-saga        |
| Re-render control     | All consumers         | Fine-grained subscriptions     |
| Best for              | Theme, auth, locale   | Complex, high-frequency state  |

---

### 6. Real-World Example — AuthContext (Login/Logout State)

```jsx
import { createContext, useContext, useState } from "react";

// Create
const AuthContext = createContext(null);

// Custom hook for convenient access
function useAuth() {
  return useContext(AuthContext);
}

// Provider — wraps the app
function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(false);

  async function login(email, password) {
    setIsLoading(true);
    // Simulate API call
    const result = await fakeLoginAPI(email, password);
    setUser(result.user);
    setIsLoading(false);
  }

  function logout() {
    setUser(null);
  }

  return (
    <AuthContext.Provider value={{ user, isLoading, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

// Usage at app root
function App() {
  return (
    <AuthProvider>
      <Router>
        <Navbar />
        <Routes />
      </Router>
    </AuthProvider>
  );
}

// Any component can access auth without prop drilling
function Navbar() {
  const { user, logout } = useAuth();

  return (
    <nav>
      {user ? (
        <>
          <span>Hello, {user.name}</span>
          <button onClick={logout}>Log Out</button>
        </>
      ) : (
        <a href="/login">Log In</a>
      )}
    </nav>
  );
}

function ProfilePage() {
  const { user } = useAuth();

  if (!user) return <p>Please log in to view your profile.</p>;
  return <p>Welcome, {user.name}. Email: {user.email}</p>;
}

function LoginForm() {
  const { login, isLoading } = useAuth();
  const [email, setEmail]     = useState("");
  const [password, setPassword] = useState("");

  return (
    <form onSubmit={e => { e.preventDefault(); login(email, password); }}>
      <input value={email}    onChange={e => setEmail(e.target.value)}    placeholder="Email" />
      <input value={password} onChange={e => setPassword(e.target.value)} type="password" placeholder="Password" />
      <button type="submit" disabled={isLoading}>
        {isLoading ? "Logging in..." : "Log In"}
      </button>
    </form>
  );
}
```

**Output**
```
Not logged in:
  Navbar: [Log In]
  ProfilePage: "Please log in to view your profile."

Submit login form → isLoading = true:
  Button: "Logging in..."

After login (user = { name: "Alice", email: "alice@dev.com" }):
  Navbar: "Hello, Alice" [Log Out]
  ProfilePage: "Welcome, Alice. Email: alice@dev.com"
  LoginForm: no longer shown (Router would redirect)
```

---

[View Interview Questions](./interview.md)
