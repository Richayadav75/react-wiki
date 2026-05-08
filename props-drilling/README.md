- Category: React Patterns
- Difficulty: Intermediate
- Related: context-api, redux, props-vs-state, useContext

### Props Drilling — The Problem and Its Solutions
**Props Drilling** (also called "prop threading") is the pattern of passing data through multiple layers of components, even when the intermediate components have no use for that data — they just carry it down to a deeply nested child.

**Analogy**
Imagine you need to pass a message to the mailroom on the 10th floor of an office building. But there's no elevator — you must hand the envelope to the receptionist on floor 1, who hands it to the manager on floor 3, who gives it to the supervisor on floor 6, who finally delivers it to the mailroom on floor 10. Floors 1, 3, and 6 don't care about the envelope — they're just middle-men. That's props drilling.

---

### 1. The Problem — Intermediate Components as Pass-Throughs

**Theory**: Props drilling itself is not a bug — it is natural in React's unidirectional data flow. It becomes a **problem** when components receive props only to pass them further down, making the code harder to maintain, refactor, and understand.

**Working Flow**
![flow-chart](flow-chart.png)

**Example — 4 levels of drilling**
```jsx
// App — owns the data
function App() {
  const user = { name: "Alice", avatar: "/alice.jpg", role: "admin" };
  return <Header user={user} />;
}

// Header — doesn't need user, just passes it on
function Header({ user }) {
  return (
    <nav>
      <Logo />
      <Navbar user={user} />   {/* passing user down */}
    </nav>
  );
}

// Navbar — doesn't need user either, still passing it
function Navbar({ user }) {
  return (
    <div>
      <NavLinks />
      <UserSection user={user} />  {/* passing user down again */}
    </div>
  );
}

// UserSection — still just a pass-through
function UserSection({ user }) {
  return <UserAvatar user={user} />;
}

// UserAvatar — finally uses the data
function UserAvatar({ user }) {
  return (
    <div>
      <img src={user.avatar} alt={user.name} />
      <span>{user.name} ({user.role})</span>
    </div>
  );
}
```

**Output**
```
Rendered HTML:
<nav>
  <Logo />
  <div>
    <NavLinks />
    <div>
      <img src="/alice.jpg" alt="Alice" />
      <span>Alice (admin)</span>
    </div>
  </div>
</nav>

Problem: Header, Navbar, UserSection all have "user" in their prop signature
         but none of them use it. If user data changes structure, all 3 break.
```

---

### 2. Why It's a Real Problem

**Theory**: Props drilling creates tight coupling between components. The symptoms grow worse as the app scales.

**Working Flow**
![flow-chart-2](flow-chart-2.png)

---

### 3. Solution 1 — Component Composition

**Theory**: Instead of passing data down, pass the component that needs the data as `children`. The parent never touches the data — it just slots in the already-configured component. This is the simplest fix and requires no extra libraries.

**Working Flow**
![flow-chart-3](flow-chart-3.png)

**Example**
```jsx
// App slots UserAvatar directly — no middlemen
function App() {
  const user = { name: "Alice", avatar: "/alice.jpg", role: "admin" };

  return (
    <Header>
      <Navbar>
        <UserAvatar user={user} />   {/* user passed ONCE, directly to user */}
      </Navbar>
    </Header>
  );
}

// Header knows nothing about user
function Header({ children }) {
  return <nav><Logo />{children}</nav>;
}

// Navbar knows nothing about user
function Navbar({ children }) {
  return <div><NavLinks />{children}</div>;
}

// UserAvatar gets user directly from App
function UserAvatar({ user }) {
  return (
    <div>
      <img src={user.avatar} alt={user.name} />
      <span>{user.name} ({user.role})</span>
    </div>
  );
}
```

**Output**
```
Same rendered HTML as before, but:
Header.props   → { children: <Navbar>...</Navbar> }  (no user prop)
Navbar.props   → { children: <UserAvatar /> }         (no user prop)
UserAvatar.props → { user: { name:"Alice", ... } }    (has user prop)

If user shape changes → only App and UserAvatar need updating
```

---

### 4. Solution 2 — Context API

**Theory**: React's Context API creates a "broadcast channel" that any component in the subtree can tune into. No manual prop passing at any level. Perfect for data that many components across different parts of the tree need.

**Working Flow**
![flow-chart-4](flow-chart-4.png)

**Example**
```jsx
import { createContext, useContext, useState } from "react";

// 1. Create the context
const UserContext = createContext(null);

// 2. Custom hook for convenience
function useUser() {
  const user = useContext(UserContext);
  if (!user) throw new Error("useUser must be inside UserContext.Provider");
  return user;
}

// 3. Provider at the top
function App() {
  const user = { name: "Alice", avatar: "/alice.jpg", role: "admin" };

  return (
    <UserContext.Provider value={user}>
      <Header />   {/* no user prop needed */}
    </UserContext.Provider>
  );
}

// Middle components — completely clean, no user prop
function Header()   { return <nav><Logo /><Navbar /></nav>; }
function Navbar()   { return <div><NavLinks /><UserAvatar /></div>; }

// Consumer — reads directly from context
function UserAvatar() {
  const user = useUser();  // zero prop drilling
  return (
    <div>
      <img src={user.avatar} alt={user.name} />
      <span>{user.name} ({user.role})</span>
    </div>
  );
}
```

**Output**
```
Header.props    → {}  (empty — no user prop at all)
Navbar.props    → {}  (empty — no user prop at all)
UserAvatar reads user from context directly

Adding user to Sidebar (new component anywhere in tree):
function Sidebar() {
  const user = useUser(); // works instantly — no changes to parent components
  return <p>Logged in as: {user.name}</p>;
}
```

---

### 5. Solution 3 — Redux / Zustand (Global State Store)

**Theory**: For complex apps with many pieces of global state that update frequently, a dedicated state management library like Redux or Zustand gives any component direct access to the state without any hierarchy at all. There is no Provider-Consumer nesting requirement.

**Working Flow**
![flow-chart-5](flow-chart-5.png)

**Example — with Redux**
```jsx
// store/userSlice.js
const userSlice = createSlice({
  name: "user",
  initialState: { name: "Alice", avatar: "/alice.jpg", role: "admin" },
  reducers: {},
});

// UserAvatar — reads directly from store, no props needed
function UserAvatar() {
  const user = useSelector(state => state.user);
  return (
    <div>
      <img src={user.avatar} alt={user.name} />
      <span>{user.name} ({user.role})</span>
    </div>
  );
}

// Any component, anywhere in the tree, can do the same
function WelcomeBanner() {
  const user = useSelector(state => state.user);
  return <h1>Welcome back, {user.name}!</h1>;
}
```

---

### 6. Side-by-Side Comparison

**Working Flow**
![flow-chart-6](flow-chart-6.png)

---

### Real-World Example — Auth Data Through 4 Layers

```text
Typical SaaS dashboard structure:
  App
    └── DashboardLayout (sidebar + header — doesn't need user)
          └── MainContent (routing wrapper — doesn't need user)
                └── ProfilePage (header with user info — needs user)
                      └── ProfileForm (editable fields — needs user)

Without solution: user drills through DashboardLayout, MainContent
With Context:     ProfilePage and ProfileForm use useContext(AuthContext)
With Redux:       ProfilePage and ProfileForm use useSelector(state => state.auth.user)
```

---

### When is Props Drilling Acceptable?

```text
OK (don't over-engineer):                  Problem (reach for a solution):
──────────────────────────────             ──────────────────────────────
2-3 component levels                       4+ component levels
1-2 props being passed                     Many props being threaded
Components are closely related             Components are unrelated
Props are actually used by all             Intermediate components don't use props
Small, focused feature                     App-wide data (auth, theme, locale)
```

---

[View Interview Questions](./interview.md)
