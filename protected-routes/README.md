- Category: React Architecture
- Difficulty: Intermediate
- Related: react-router, context-api, hooks

### Protected Routes in React
A protected route is a route that only renders its content when the user is authenticated (and optionally authorized with the right role). If the check fails, the user is redirected — usually to the login page — instead of seeing the content.

**Analogy**
A hotel corridor with keycards. The lobby (public routes) is open to anyone. The room doors (protected routes) require a valid keycard (auth token). When you try to open a door without a keycard, a security guard (ProtectedRoute component) escorts you back to reception (login page). A VIP floor (admin routes) requires a keycard AND VIP status (role check).

---

### 1. How React Router v6 Routing Works
**Theory**: React Router v6 uses `<Routes>` and `<Route>` to map URL paths to components. A nested `<Route>` without a `path` acts as a layout wrapper. `<Outlet />` renders the matched child route inside the wrapper — this is the foundation of the Outlet pattern for protected routes.

**Working Flow**
![flow-chart](flow-chart.png)

**Example — Basic Router Setup**
```jsx
import { BrowserRouter, Routes, Route, Navigate, Outlet, Link } from "react-router-dom";

function App() {
  return (
    <BrowserRouter>
      <Routes>
        {/* Public routes — anyone can visit */}
        <Route path="/"       element={<Home />} />
        <Route path="/login"  element={<Login />} />
        <Route path="/signup" element={<Signup />} />

        {/* Protected routes — wrapped in ProtectedRoute layout */}
        <Route element={<ProtectedRoute />}>
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/profile"   element={<Profile />} />
          <Route path="/settings"  element={<Settings />} />
        </Route>

        {/* Admin-only routes */}
        <Route element={<ProtectedRoute requiredRole="admin" />}>
          <Route path="/admin"       element={<AdminPanel />} />
          <Route path="/admin/users" element={<AdminUsers />} />
        </Route>

        {/* 404 fallback */}
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
}
```

**Output**
```
/ (home)         → <Home />                      (public, always)
/login           → <Login />                     (public, always)
/dashboard       → ProtectedRoute check → pass → <Dashboard />
/dashboard       → ProtectedRoute check → fail → redirect to /login
/admin           → ProtectedRoute + role check → admin → <AdminPanel />
/admin           → ProtectedRoute + role check → user  → redirect to /forbidden
```

---

### 2. The ProtectedRoute Component (Outlet Pattern)
**Theory**: `ProtectedRoute` is a wrapper component — it doesn't render its own UI. It reads auth state and either renders `<Outlet />` (which renders the actual child route) or redirects. The `replace` prop on `<Navigate>` replaces the history entry so the Back button doesn't loop.

**Working Flow**
![flow-chart-2](flow-chart-2.png)

**Example**
```jsx
import { Navigate, Outlet, useLocation } from "react-router-dom";
import { useAuth } from "./context/AuthContext";

function ProtectedRoute({ requiredRole }) {
  const { user, isAuthenticated } = useAuth(); // from AuthContext
  const location = useLocation();              // current URL

  // Not logged in → go to login, remember where they were
  if (!isAuthenticated) {
    return <Navigate to="/login" state={{ from: location }} replace />;
    //                                  ↑ pass current URL in router state
    //                                    so Login can redirect back after success
  }

  // Logged in but wrong role → forbidden page
  if (requiredRole && user?.role !== requiredRole) {
    return <Navigate to="/forbidden" replace />;
  }

  // All checks pass → render the child route
  return <Outlet />;
}

export default ProtectedRoute;
```

**Output**
```
Not logged in, visits /dashboard:
  → Navigate to="/login" state={{ from: { pathname: "/dashboard" } }}
  → After login, Login component can redirect back to /dashboard

Logged in as user, visits /admin:
  → requiredRole="admin", user.role="user" → mismatch
  → Navigate to="/forbidden"

Logged in as admin, visits /admin:
  → all checks pass → <Outlet /> renders <AdminPanel />
```

---

### 3. Auth Context — Sharing Auth State Without Prop Drilling
**Theory**: Authentication state (user object, token, login/logout functions) needs to be accessible in `ProtectedRoute`, the navbar, and any component that checks "is this my content?". Context API is the right tool — it broadcasts auth state to the whole tree.

**Working Flow**
![flow-chart-3](flow-chart-3.png)

**Example**
```jsx
// context/AuthContext.jsx
import { createContext, useContext, useState, useEffect } from "react";

const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);

  // Persist auth: re-hydrate from localStorage on app load
  useEffect(() => {
    const token    = localStorage.getItem("token");
    const userData = localStorage.getItem("user");
    if (token && userData) {
      setUser(JSON.parse(userData)); // restore session
    }
  }, []);

  function login(userData, token) {
    localStorage.setItem("token", token);
    localStorage.setItem("user", JSON.stringify(userData));
    setUser(userData);
  }

  function logout() {
    localStorage.removeItem("token");
    localStorage.removeItem("user");
    setUser(null);
  }

  const value = {
    user,
    isAuthenticated: !!user,   // truthy shorthand
    login,
    logout,
  };

  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
}

// Custom hook for easy consumption
export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) throw new Error("useAuth must be used inside AuthProvider");
  return context;
}
```

**Output**
```
Page refresh with valid localStorage:
  useEffect runs → finds token + user → setUser(parsedUser)
  → isAuthenticated = true → ProtectedRoutes render normally ✓

logout() called:
  → localStorage cleared → user = null → isAuthenticated = false
  → ProtectedRoutes redirect to /login ✓
```

---

### 4. Login Page — Redirect After Login
**Theory**: When the user was redirected from a protected route, the original URL is stored in React Router's `location.state`. After successful login, redirect back there instead of always going to `/dashboard`.

**Working Flow**
![flow-chart-4](flow-chart-4.png)

**Example**
```jsx
import { useNavigate, useLocation, Link } from "react-router-dom";
import { useAuth } from "./context/AuthContext";

function Login() {
  const [email, setEmail]       = useState("");
  const [password, setPassword] = useState("");
  const [error, setError]       = useState(null);
  const [loading, setLoading]   = useState(false);

  const { login }  = useAuth();
  const navigate   = useNavigate();
  const location   = useLocation();

  // Where to go after login — default to /dashboard
  const from = location.state?.from?.pathname || "/dashboard";

  async function handleSubmit(e) {
    e.preventDefault();
    setError(null);
    setLoading(true);
    try {
      const res  = await fetch("/api/auth/login", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ email, password }),
      });
      if (!res.ok) throw new Error("Invalid credentials");
      const { user, token } = await res.json();

      login(user, token);            // update context + localStorage
      navigate(from, { replace: true }); // go back to original destination
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <h2>Sign In</h2>
      {error && <p style={{ color: "red" }}>{error}</p>}
      <input type="email"    value={email}    onChange={e => setEmail(e.target.value)}    required />
      <input type="password" value={password} onChange={e => setPassword(e.target.value)} required />
      <button type="submit" disabled={loading}>
        {loading ? "Signing in..." : "Sign In"}
      </button>
    </form>
  );
}
```

**Output**
```
Visited /settings (not logged in):
  → Login page, state.from = /settings

Submit valid credentials:
  → "Signing in..."
  → login(user, token) → context updated, localStorage set
  → navigate("/settings", replace: true) ← back to original destination

Submit invalid credentials:
  → "Invalid credentials" error message
```

---

### 5. Role-Based Access Control (RBAC)
**Theory**: Some routes should only be accessible to specific roles (admin, moderator, editor). Extend `ProtectedRoute` with a `requiredRole` prop. For complex apps, use an array of allowed roles.

**Working Flow**
![flow-chart-5](flow-chart-5.png)

**Example**
```jsx
// Flexible RBAC ProtectedRoute
function ProtectedRoute({ allowedRoles }) {
  const { user, isAuthenticated } = useAuth();
  const location = useLocation();

  if (!isAuthenticated) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  // Role check — allowedRoles is optional (omit to allow any logged-in user)
  if (allowedRoles && !allowedRoles.includes(user.role)) {
    return <Navigate to="/forbidden" replace />;
  }

  return <Outlet />;
}

// App.jsx usage
<Routes>
  <Route element={<ProtectedRoute />}>
    <Route path="/dashboard" element={<Dashboard />} />  {/* any logged-in user */}
  </Route>

  <Route element={<ProtectedRoute allowedRoles={["admin"]} />}>
    <Route path="/admin" element={<AdminPanel />} />     {/* admin only */}
  </Route>

  <Route element={<ProtectedRoute allowedRoles={["admin", "editor"]} />}>
    <Route path="/editor" element={<EditorDashboard />} /> {/* admin or editor */}
  </Route>

  <Route path="/forbidden" element={<Forbidden />} />
</Routes>
```

**Output**
```
user.role = "user":
  /dashboard → pass (any logged-in)
  /admin     → fail → /forbidden
  /editor    → fail → /forbidden

user.role = "editor":
  /dashboard → pass
  /admin     → fail → /forbidden
  /editor    → pass

user.role = "admin":
  /dashboard → pass
  /admin     → pass
  /editor    → pass (admin is in allowedRoles)
```

---

### 6. Persisting Auth State Across Page Refresh
**Theory**: React state is lost on page refresh. To keep the user logged in, store the token (and optionally the user object) in `localStorage` or a secure `httpOnly` cookie. On app load, `useEffect` in `AuthProvider` re-hydrates state from storage.

**Working Flow**
![flow-chart-6](flow-chart-6.png)

**Example — Token Expiry Handling**
```jsx
// apiFetch.js — global fetch wrapper that handles 401
import { useAuth } from "./context/AuthContext";

// Standalone utility (not a hook) — use events to communicate with AuthContext
async function apiFetch(url, options = {}) {
  const token = localStorage.getItem("token");

  const res = await fetch(url, {
    ...options,
    headers: {
      "Content-Type": "application/json",
      ...(token ? { Authorization: `Bearer ${token}` } : {}),
      ...options.headers,
    },
  });

  if (res.status === 401) {
    // Token expired — clear auth and redirect
    localStorage.removeItem("token");
    localStorage.removeItem("user");
    window.dispatchEvent(new Event("auth:logout")); // signal AuthProvider
    throw new Error("Session expired. Please log in again.");
  }

  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
}

// AuthProvider listens for the logout event
useEffect(() => {
  function handleLogout() { setUser(null); }
  window.addEventListener("auth:logout", handleLogout);
  return () => window.removeEventListener("auth:logout", handleLogout);
}, []);
```

**Output**
```
Token valid:
  apiFetch("/api/profile") → 200 → { name: "Richa", ... }

Token expired (after 24h):
  apiFetch("/api/profile") → 401
  → localStorage cleared
  → "auth:logout" event fired → AuthContext sets user = null
  → ProtectedRoute renders <Navigate to="/login" />
  → User sees login page with "Session expired..." toast
```

---

### Real-World Example: /dashboard Protected, /admin with Role Check
```jsx
// Full working setup

// index.jsx
import { AuthProvider } from "./context/AuthContext";
<AuthProvider><App /></AuthProvider>

// App.jsx
function App() {
  return (
    <BrowserRouter>
      <Navbar />
      <Routes>
        <Route path="/"       element={<Home />} />
        <Route path="/login"  element={<Login />} />
        <Route path="/forbidden" element={<Forbidden />} />

        {/* Protected: any logged-in user */}
        <Route element={<ProtectedRoute />}>
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/profile"   element={<Profile />} />
        </Route>

        {/* Protected: admin only */}
        <Route element={<ProtectedRoute allowedRoles={["admin"]} />}>
          <Route path="/admin"          element={<AdminLayout />}>
            <Route index             element={<AdminDashboard />} />
            <Route path="users"      element={<AdminUsers />} />
            <Route path="analytics"  element={<AdminAnalytics />} />
          </Route>
        </Route>

        <Route path="*" element={<Navigate to="/" />} />
      </Routes>
    </BrowserRouter>
  );
}

// Navbar — shows different links based on auth + role
function Navbar() {
  const { user, isAuthenticated, logout } = useAuth();
  const navigate = useNavigate();

  return (
    <nav>
      <Link to="/">Home</Link>
      {isAuthenticated ? (
        <>
          <Link to="/dashboard">Dashboard</Link>
          {user?.role === "admin" && <Link to="/admin">Admin</Link>}
          <button onClick={() => { logout(); navigate("/login"); }}>Logout</button>
        </>
      ) : (
        <Link to="/login">Login</Link>
      )}
    </nav>
  );
}
```

**Output**
```
Not logged in:
  Navbar: [Home] [Login]
  /dashboard → redirected to /login

Logged in as user (role="user"):
  Navbar: [Home] [Dashboard] [Logout]
  /dashboard → Dashboard renders ✓
  /admin     → redirected to /forbidden

Logged in as admin (role="admin"):
  Navbar: [Home] [Dashboard] [Admin] [Logout]
  /dashboard → Dashboard renders ✓
  /admin     → AdminPanel renders ✓
  /admin/users → AdminUsers renders ✓

IMPORTANT: Client-side protection = UX only.
Every API endpoint MUST verify the JWT token server-side.
```

---

[View Interview Questions](./interview.md)
