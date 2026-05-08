- Category: React Hooks
- Difficulty: Intermediate
- Related: context-api, useReducer, hooks, props-drilling

### useContext — Consume Context Without Wrapper Components
`useContext` is a React hook that reads a value from a React context directly inside a functional component. It eliminates the verbose `<Context.Consumer>` JSX wrapper and makes consuming global values (theme, auth, locale) clean and readable.

**Analogy**
A company-wide announcement system. Instead of the CEO whispering the company policy to a manager, who tells a team lead, who tells each employee (prop drilling) — the CEO broadcasts it on the intercom (Provider) and every employee who has the app installed (useContext) hears it directly, wherever they are in the building.

---

### 1. Creating and Providing Context

**Theory**
Context has two parts:
1. `createContext(defaultValue)` — creates a context object
2. `<Context.Provider value={...}>` — broadcasts a value to all descendants

Any component inside the Provider can read that value using `useContext`.

**Working Flow**

![flow-chart](flow-chart.png)

**Example**
```jsx
import { createContext, useContext } from 'react';

// Step 1 — Create the context
const ThemeContext = createContext('light');

// Step 2 — Provide a value at the top level
function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Header />
      <Main />
    </ThemeContext.Provider>
  );
}

// Step 3 — Consume in any descendant (no matter how deep)
function Header() {
  const theme = useContext(ThemeContext);
  return (
    <header style={{ background: theme === 'dark' ? '#333' : '#fff' }}>
      Theme: {theme}
    </header>
  );
}

function Main() {
  return (
    <div>
      <Sidebar />
    </div>
  );
}

function Sidebar() {
  const theme = useContext(ThemeContext);
  return <aside className={theme}>Sidebar</aside>;
}
```

**Output**
```
// App renders with value="dark":
Header shows → background: #333, text: "Theme: dark"
Sidebar has  → className="dark"
```

---

### 2. useContext vs Prop Drilling

**Theory**
Without context, data must be passed as props through every intermediate component — even ones that don't use it. Context skips the middle components entirely.

**Working Flow**

![flow-chart-2](flow-chart-2.png)

**Example**
```jsx
// ❌ Prop drilling — user passed through 3 layers unnecessarily
function App() {
  const user = { name: 'Richa', role: 'Admin' };
  return <Layout user={user} />;
}
function Layout({ user }) {
  return <Sidebar user={user} />;  // Layout doesn't use user
}
function Sidebar({ user }) {
  return <UserBadge user={user} />; // Sidebar doesn't use user
}
function UserBadge({ user }) {
  return <p>{user.name} — {user.role}</p>; // only this needs it
}

// ✅ Context — UserBadge reads directly, no intermediate passing
const UserContext = createContext(null);

function App() {
  const user = { name: 'Richa', role: 'Admin' };
  return (
    <UserContext.Provider value={user}>
      <Layout />
    </UserContext.Provider>
  );
}
function Layout()  { return <Sidebar />; }
function Sidebar() { return <UserBadge />; }
function UserBadge() {
  const user = useContext(UserContext);
  return <p>{user.name} — {user.role}</p>;
}
```

**Output**
```
// Both approaches render the same output:
Richa — Admin

// But with context, Layout and Sidebar are cleaner — no user prop
```

---

### 3. Dynamic Context — State + Context Together

**Theory**
To make context values dynamic (changeable at runtime), pair the Provider with `useState` or `useReducer`. The value updates propagate instantly to all consumers when state changes.

**Working Flow**

![flow-chart-3](flow-chart-3.png)

**Example**
```jsx
import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext(null);

// Provider manages the state
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  function toggleTheme() {
    setTheme(t => t === 'light' ? 'dark' : 'light');
  }

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// Consumer reads and can change the theme
function ThemedButton() {
  const { theme, toggleTheme } = useContext(ThemeContext);

  return (
    <button
      onClick={toggleTheme}
      style={{
        background: theme === 'dark' ? '#222' : '#eee',
        color:      theme === 'dark' ? '#fff' : '#000',
      }}
    >
      Current theme: {theme} — Click to toggle
    </button>
  );
}

function App() {
  return (
    <ThemeProvider>
      <ThemedButton />
      <ThemedButton />
    </ThemeProvider>
  );
}
```

**Output**
```
// Initial:
Both buttons → light background, dark text, "Current theme: light"

// Click either button:
Both buttons → dark background, white text, "Current theme: dark"
(both update because they share the same context value)
```

---

### 4. Multiple Contexts

**Theory**
An app typically has several independent concerns (theme, auth, locale). Each gets its own context. Consumers only subscribe to the contexts they actually need — no performance impact from unrelated context updates.

**Working Flow**

![flow-chart-4](flow-chart-4.png)

**Example**
```jsx
import { createContext, useContext, useState } from 'react';

const AuthContext  = createContext(null);
const ThemeContext = createContext('light');

function AuthProvider({ children }) {
  const [user, setUser] = useState(null);

  function login(name) {
    setUser({ name, role: 'user' });
  }
  function logout() {
    setUser(null);
  }

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

// Custom hooks for clean consumption
const useAuth  = () => useContext(AuthContext);
const useTheme = () => useContext(ThemeContext);

function Navbar() {
  const { user, logout } = useAuth();
  const theme            = useTheme();

  return (
    <nav style={{ background: theme === 'dark' ? '#111' : '#f5f5f5' }}>
      {user ? (
        <>
          <span>Hello, {user.name}!</span>
          <button onClick={logout}>Logout</button>
        </>
      ) : (
        <span>Please log in</span>
      )}
    </nav>
  );
}

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <AuthProvider>
        <Navbar />
      </AuthProvider>
    </ThemeContext.Provider>
  );
}
```

**Output**
```
// Initial (no user):
Navbar → dark bg, "Please log in"

// After login("Richa"):
Navbar → dark bg, "Hello, Richa! [Logout]"
```

---

### 5. Context Re-render Behavior

**Theory**
Every component that calls `useContext(MyContext)` re-renders whenever `MyContext`'s value changes — even if the specific property the component uses didn't change. To optimize, split context by concern, or wrap the value in `useMemo`.

**Working Flow**

![flow-chart-5](flow-chart-5.png)

**Example**
```jsx
import { createContext, useContext, useState, useMemo } from 'react';

const StoreContext = createContext(null);

function StoreProvider({ children }) {
  const [count, setCount] = useState(0);
  const [name, setName]   = useState('Shop');

  // useMemo prevents a new object on every render → only updates when values change
  const value = useMemo(
    () => ({ count, setCount, name, setName }),
    [count, name]
  );

  return (
    <StoreContext.Provider value={value}>
      {children}
    </StoreContext.Provider>
  );
}
```

---

### 6. Real-World — Auth Context

**Working Flow**

![flow-chart-6](flow-chart-6.png)

**Example**
```jsx
import { createContext, useContext, useState, useEffect } from 'react';

const AuthContext = createContext(null);

function AuthProvider({ children }) {
  const [user, setUser]         = useState(null);
  const [loading, setLoading]   = useState(true);

  // Restore session from localStorage on app start
  useEffect(() => {
    const saved = localStorage.getItem('user');
    if (saved) setUser(JSON.parse(saved));
    setLoading(false);
  }, []);

  function login(userData) {
    setUser(userData);
    localStorage.setItem('user', JSON.stringify(userData));
  }

  function logout() {
    setUser(null);
    localStorage.removeItem('user');
  }

  return (
    <AuthContext.Provider value={{ user, loading, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

// Hook for any component to access auth
function useAuth() {
  const context = useContext(AuthContext);
  if (!context) throw new Error('useAuth must be inside AuthProvider');
  return context;
}

// Usage in components
function ProfilePage() {
  const { user, logout } = useAuth();
  return (
    <div>
      <h2>Welcome, {user.name}!</h2>
      <button onClick={logout}>Logout</button>
    </div>
  );
}
```

---

[View Interview Questions](./interview.md)
