# Props Drilling — Interview Questions

---

**1. What is props drilling and when does it become a problem?**

Props drilling is passing data through multiple layers of components — including ones that don't use the data — to reach a deeply nested child. It is fine for 2-3 levels. It becomes a problem when intermediate components accumulate props they don't use, making refactoring painful and components harder to reuse.

```jsx
// Props drilling — Header and Navbar carry "user" they don't need
function App()    { return <Header user={user} />; }
function Header({ user }) { return <Navbar user={user} />; }  // doesn't use user
function Navbar({ user }) { return <UserAvatar user={user} />; }  // doesn't use user
function UserAvatar({ user }) { return <img src={user.avatar} />; } // USES user
```

---

**2. What is the simplest solution to props drilling without any extra library?**

Component composition — pass the deeply nested component as `children`. The parent provides the data directly when composing, so intermediate components receive no data-related props at all.

```jsx
// App composes UserAvatar with user — no drilling
function App() {
  const user = { name: "Alice", avatar: "/alice.jpg" };
  return (
    <Header>
      <Navbar>
        <UserAvatar user={user} />
      </Navbar>
    </Header>
  );
}

// Clean intermediate components
function Header({ children }) { return <nav>{children}</nav>; }
function Navbar({ children }) { return <div>{children}</div>; }

// UserAvatar receives user once, directly from App
function UserAvatar({ user }) { return <img src={user.avatar} alt={user.name} />; }
```

---

**3. How does the Context API solve props drilling?**

Context creates a value channel that any component in the subtree can access directly via `useContext`, without any prop passing at intermediate levels.

```jsx
const UserContext = createContext(null);

function App() {
  const user = { name: "Alice", avatar: "/alice.jpg" };
  return (
    <UserContext.Provider value={user}>
      <Header />   {/* no user prop */}
    </UserContext.Provider>
  );
}

// Intermediate components — completely clean
function Header() { return <nav><Navbar /></nav>; }
function Navbar() { return <div><UserAvatar /></div>; }

// Consumer reads directly, skipping all layers
function UserAvatar() {
  const user = useContext(UserContext);
  return <img src={user.avatar} alt={user.name} />;
}
```

---

**4. What is the downside of using Context API for all global state?**

Context re-renders every component that calls `useContext` whenever the context value changes — even if the specific value that component reads didn't change. For high-frequency updates this causes performance problems.

```jsx
// Problem: both Avatar and CartBadge re-render whenever ANY of appState changes
const AppContext = createContext();
// { user, cart, notifications, theme } — all in one context

function UserAvatar() {
  const { user } = useContext(AppContext); // re-renders on cart update too!
}

// Fix: split into separate contexts
const UserContext = createContext();
const CartContext = createContext();

function UserAvatar() {
  const user = useContext(UserContext); // only re-renders when user changes
}
function CartBadge() {
  const cart = useContext(CartContext); // only re-renders when cart changes
}
```

---

**5. When should you use Redux vs Context API to solve props drilling?**

```text
Use Context when:                       Use Redux when:
─────────────────────────────           ─────────────────────────────
Auth user, theme, locale               Cart, orders, filters, search
State changes infrequently             State changes on every user action
Simple read, rare write                Complex reducers / derived state
Small to medium project                Large team / large codebase
No DevTools needed                     Need time-travel debugging
```

```jsx
// Context — good for theme (changes rarely)
const ThemeContext = createContext("light");
function Button() {
  const theme = useContext(ThemeContext);
  return <button className={theme}>Click</button>;
}

// Redux — good for cart (changes on every add/remove)
function CartBadge() {
  const count = useSelector(state => state.cart.items.length);
  return <span>{count}</span>;
}
```

---

**6. Does props drilling affect performance?**

Yes, indirectly. Intermediate components that receive props they don't use still re-render when those props change (because their parent re-renders and passes new prop references). React.memo can help but it doesn't solve the root design problem.

```jsx
// Problem: Header re-renders whenever user changes, even though it doesn't use user
function Header({ user }) {  // carries user only to pass it down
  console.log("Header re-rendered"); // fires on every user update
  return <Navbar user={user} />;
}

// React.memo partial fix
const Header = React.memo(function Header({ user }) {
  return <Navbar user={user} />;
});
// Still re-renders when user object reference changes (which happens on every setState)

// Real fix: remove user from Header's props using composition or context
```

---

**7. What is "prop threading" and how is it different from props drilling?**

They are the same pattern — different names. "Prop threading" is sometimes used to emphasize that props are being threaded through a long chain of components, like threading a needle through fabric layers. Both describe intermediate components carrying props they don't consume.

```text
Props drilling    = the problem name used in interviews and articles
Prop threading    = descriptive metaphor (threading the data through layers)
Prop plumbing     = another informal synonym

All three describe:
  Parent has data → must pass through N intermediate components → deep child uses it
```

---

**8. Can you show a real-world example of props drilling with auth data across 4 layers?**

```jsx
// 4-level drill: App → DashboardLayout → MainContent → ProfilePage → ProfileForm

// App — owns auth data
function App() {
  const [authUser, setAuthUser] = useState({ name: "Alice", email: "alice@co.com" });
  return <DashboardLayout authUser={authUser} setAuthUser={setAuthUser} />;
}

// DashboardLayout — doesn't use authUser, just passes it
function DashboardLayout({ authUser, setAuthUser }) {
  return (
    <div className="layout">
      <Sidebar />
      <MainContent authUser={authUser} setAuthUser={setAuthUser} />
    </div>
  );
}

// MainContent — doesn't use authUser, just passes it
function MainContent({ authUser, setAuthUser }) {
  return <ProfilePage authUser={authUser} setAuthUser={setAuthUser} />;
}

// ProfilePage — finally uses authUser
function ProfilePage({ authUser, setAuthUser }) {
  return (
    <div>
      <h1>Profile: {authUser.name}</h1>
      <ProfileForm authUser={authUser} setAuthUser={setAuthUser} />
    </div>
  );
}

// Solution with Context — eliminate drilling entirely
const AuthContext = createContext();
function App() {
  const [authUser, setAuthUser] = useState({ name: "Alice", email: "alice@co.com" });
  return (
    <AuthContext.Provider value={{ authUser, setAuthUser }}>
      <DashboardLayout />   {/* no props needed */}
    </AuthContext.Provider>
  );
}
// DashboardLayout, MainContent, ProfilePage — all have empty prop signatures
// ProfileForm reads: const { authUser, setAuthUser } = useContext(AuthContext);
```

---

**9. What is the "render props" pattern and how does it relate to props drilling?**

Render props was a pre-hooks technique to share stateful logic. It is an alternative to props drilling that moves logic up and injects it via a function prop. Custom Hooks have largely replaced it.

```jsx
// Render props (older pattern)
function WithUser({ render }) {
  const user = { name: "Alice" }; // stateful logic here
  return render(user);            // injects user via function
}

// Usage — no prop drilling through intermediates
function Header() {
  return <WithUser render={(user) => <UserAvatar user={user} />} />;
}

// Modern equivalent — custom hook
function useCurrentUser() {
  return { name: "Alice" }; // logic here
}

function UserAvatar() {
  const user = useCurrentUser(); // reads directly — no drilling, no wrapper
  return <img src={user.avatar} />;
}
```

---

**10. How do you decide which solution to use for props drilling?**

```text
Decision tree:

Prop only travels 1-2 levels?
  → Keep the prop as-is. Don't over-engineer.

Multiple unrelated components need the same data?
  → Context API (auth, theme, locale)

State updates are complex or high-frequency?
  → Redux or Zustand

Components are laid out in a clear slot pattern (header/sidebar/content)?
  → Component composition with children

Data is server state (fetched from API)?
  → React Query or SWR (avoids the problem entirely by fetching where needed)
```
