# Protected Routes — Interview Questions

---

**1. What is a Protected Route and why do we need it?**

A protected route is a route that requires the user to be authenticated (and optionally authorized with a role) before rendering. Without it, anyone could navigate directly to `/dashboard` or `/admin` by typing the URL.

```text
Public route  → /home, /login, /signup     → accessible to everyone
Protected route → /dashboard, /profile    → authenticated users only
Admin route   → /admin, /admin/users      → specific roles only
```

Implementation: a wrapper component that checks auth state and either renders `<Outlet />` (child route) or redirects with `<Navigate />`.

---

**2. How does the `<Outlet />` pattern work for protected routes in React Router v6?**

`<Outlet />` renders the matched child route inside a parent layout. In protected routes, the parent contains the auth check — if it passes, `<Outlet />` renders the actual page.

```jsx
function ProtectedRoute() {
  const { isAuthenticated } = useAuth();

  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }

  return <Outlet />; // renders <Dashboard />, <Profile />, etc. based on URL
}

// Usage — ONE wrapper, MANY protected routes
<Route element={<ProtectedRoute />}>
  <Route path="/dashboard" element={<Dashboard />} />
  <Route path="/profile"   element={<Profile />} />
  <Route path="/settings"  element={<Settings />} />
</Route>
```

Without `<Outlet />`, you'd need to wrap each route individually — far more boilerplate.

---

**3. What is the purpose of `replace` on `<Navigate replace />`?**

Without `replace`, the redirect adds a new entry to the browser history stack. The user clicks Back → lands on the protected page again → gets redirected again — infinite redirect loop.

```jsx
// WITHOUT replace — history stack:
// /dashboard → /login → Back → /dashboard → /login (loop!)

// WITH replace — history stack:
// /login  (the /dashboard entry is replaced, not added)
// → Back button goes to the page before /dashboard, no loop

return <Navigate to="/login" replace />;
```

---

**4. How do you redirect the user back to where they were after login?**

Pass the current location in router state when redirecting to login. After successful login, navigate to `state.from`.

```jsx
// ProtectedRoute — remember where the user was
const location = useLocation();
return <Navigate to="/login" state={{ from: location }} replace />;

// Login component — redirect back after success
function Login() {
  const location = useLocation();
  const navigate = useNavigate();
  const { login } = useAuth();

  const from = location.state?.from?.pathname || "/dashboard";

  async function handleSubmit(e) {
    e.preventDefault();
    const { user, token } = await authenticateUser(email, password);
    login(user, token);
    navigate(from, { replace: true }); // ← go back to /settings, /profile, etc.
  }
}
```

---

**5. How do you implement role-based access control (RBAC) in protected routes?**

Add an `allowedRoles` prop to `ProtectedRoute`. If the user's role is not in the array, redirect to `/forbidden`.

```jsx
function ProtectedRoute({ allowedRoles }) {
  const { user, isAuthenticated } = useAuth();
  const location = useLocation();

  if (!isAuthenticated) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  if (allowedRoles && !allowedRoles.includes(user.role)) {
    return <Navigate to="/forbidden" replace />;
  }

  return <Outlet />;
}

// Route setup
<Route element={<ProtectedRoute />}>
  <Route path="/dashboard" element={<Dashboard />} />      // any user
</Route>

<Route element={<ProtectedRoute allowedRoles={["admin"]} />}>
  <Route path="/admin" element={<AdminPanel />} />         // admin only
</Route>

<Route element={<ProtectedRoute allowedRoles={["admin", "editor"]} />}>
  <Route path="/editor" element={<EditorDashboard />} />   // admin or editor
</Route>
```

---

**6. Where should auth state be stored and why?**

Auth state belongs in React Context (for in-memory access across components) AND in `localStorage` or a secure cookie (to survive page refresh).

```jsx
// Context — fast, reactive, available to any component
const { user, isAuthenticated } = useAuth();  // immediate, no re-render needed

// localStorage — persists across refresh
localStorage.setItem("token", jwt);
localStorage.setItem("user", JSON.stringify(userObject));

// On app load — re-hydrate from localStorage
useEffect(() => {
  const token = localStorage.getItem("token");
  const user  = localStorage.getItem("user");
  if (token && user) {
    setUser(JSON.parse(user)); // restore session silently
  }
}, []);
```

Security note: `localStorage` is vulnerable to XSS. For high-security apps, use `httpOnly` cookies (not accessible via JavaScript) and validate tokens server-side.

---

**7. Is client-side route protection enough for security?**

No. Client-side protection is UX only — it hides UI from unauthorized users. A technically savvy user can bypass it using browser dev tools or by calling your API directly.

```text
Client-side protection:
  ✓ Good UX — redirects unauthorized users
  ✗ Security — user can open DevTools, set a fake token in localStorage, bypass redirect

Real security — ALWAYS check on the server:
  Every API endpoint must:
  1. Read the Authorization: Bearer <token> header
  2. Verify the JWT signature and expiry
  3. Check the user's role for the resource
  4. Return 401/403 if invalid — regardless of what the UI says
```

```javascript
// Server-side check (Express example)
app.get("/api/admin/users", verifyJWT, requireRole("admin"), (req, res) => {
  res.json(users); // only reaches here if JWT valid AND role = "admin"
});
```

---

**8. How do you handle a token expiring while the user is on a protected page?**

Create a global API wrapper that detects 401 responses and logs the user out automatically.

```jsx
// api.js
async function apiFetch(url, options = {}) {
  const token = localStorage.getItem("token");

  const res = await fetch(url, {
    ...options,
    headers: {
      Authorization: `Bearer ${token}`,
      "Content-Type": "application/json",
      ...options.headers,
    },
  });

  if (res.status === 401) {
    // Token expired — log out and redirect
    localStorage.removeItem("token");
    localStorage.removeItem("user");
    window.dispatchEvent(new Event("auth:logout")); // AuthProvider listens
    throw new Error("Session expired");
  }

  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
}

// AuthProvider listens and clears state
useEffect(() => {
  const handleLogout = () => setUser(null);
  window.addEventListener("auth:logout", handleLogout);
  return () => window.removeEventListener("auth:logout", handleLogout);
}, []);
```

---

**9. How do you protect nested routes (e.g., `/admin/users`, `/admin/analytics`)?**

Nest child routes under a protected parent. The single `ProtectedRoute` wrapper applies to all children.

```jsx
<Route element={<ProtectedRoute allowedRoles={["admin"]} />}>
  <Route path="/admin" element={<AdminLayout />}>       // AdminLayout has its own <Outlet />
    <Route index          element={<AdminDashboard />} /> // /admin
    <Route path="users"   element={<AdminUsers />} />    // /admin/users
    <Route path="reports" element={<AdminReports />} />  // /admin/reports
  </Route>
</Route>
```

```jsx
// AdminLayout.jsx
function AdminLayout() {
  return (
    <div>
      <AdminSidebar />
      <main>
        <Outlet /> {/* renders AdminDashboard, AdminUsers, or AdminReports */}
      </main>
    </div>
  );
}
```

One auth check protects all three sub-routes. `AdminLayout` provides the shared sidebar.

---

**10. What is the difference between authentication and authorization?**

```text
Authentication → "Who are you?"
  → Login with email + password
  → Verify credentials → issue JWT token
  → user is now "authenticated"

Authorization → "What are you allowed to do?"
  → User visits /admin
  → Server checks: is user.role === "admin"?
  → If yes → access granted
  → If no  → 403 Forbidden

Example flow:
  1. POST /api/login → 200 OK → { token, user: { id, name, role: "editor" } }
  2. GET /api/admin/users (with token) → 403 Forbidden (authenticated but not authorized)
  3. GET /api/editor/posts (with token) → 200 OK (authenticated and authorized)
```

In React protected routes:
- `isAuthenticated` check = authentication gate
- `allowedRoles` check = authorization gate

Both must pass. The server must enforce both independently of the client.
