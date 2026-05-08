# Context API Interview Questions

---

**1. What problem does the Context API solve?**

It solves prop drilling — the need to pass data through multiple intermediate components that do not use the data themselves, only forwarding it deeper. Context lets any component in the tree read a value directly without receiving it as a prop.

```
WITHOUT Context:
  App (user) → Layout (user) → Sidebar (user) → Avatar (user)
  Layout and Sidebar must accept user even though they never use it.

WITH Context:
  App (provides user via Provider)
  Avatar calls useContext(UserContext) → gets user directly
  Layout and Sidebar are not affected.
```

---

**2. What are the three parts of the Context API?**

1. `createContext(defaultValue)` — creates the context object.
2. `Context.Provider` — wraps part of the tree and supplies the value.
3. `useContext(Context)` — reads the value inside any descendant component.

```jsx
const ThemeContext = createContext("light");          // 1. create

<ThemeContext.Provider value="dark">                 // 2. provide
  <App />
</ThemeContext.Provider>

function Card() {
  const theme = useContext(ThemeContext); // 3. consume → "dark"
}
```

---

**3. What is the default value of createContext used for?**

It is the fallback value when a component calls `useContext` but has no matching Provider anywhere above it in the tree. In most real apps there is always a Provider, so the default acts as a safety net for testing or edge cases.

```jsx
const ThemeContext = createContext("light"); // default = "light"

// If Card is rendered outside any ThemeContext.Provider:
function Card() {
  const theme = useContext(ThemeContext); // → "light" (default, not from Provider)
}
```

---

**4. How do you make a context value dynamic (changeable)?**

Combine context with `useState` in the Provider component. Pass both the current value and the setter function inside the Provider's `value` prop.

```jsx
const ThemeContext = createContext({ theme: "light", setTheme: () => {} });

function App() {
  const [theme, setTheme] = useState("light");

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <App />
    </ThemeContext.Provider>
  );
}

// Any consumer can now both read and update:
function ToggleButton() {
  const { theme, setTheme } = useContext(ThemeContext);
  return (
    <button onClick={() => setTheme(t => t === "light" ? "dark" : "light")}>
      {theme}
    </button>
  );
}
```

---

**5. What is the performance drawback of Context API?**

Every component that calls `useContext(SomeContext)` re-renders whenever the context value changes — even if the specific part it reads did not change. This can be amplified when the Provider wraps a new object literal on every render.

```jsx
// PROBLEM — new object on every render triggers all consumers
<Context.Provider value={{ user, theme }}>  // new object reference each time

// FIX — memoize the value
const value = useMemo(() => ({ user, theme }), [user, theme]);
<Context.Provider value={value}>
```

---

**6. Can you have multiple Providers in one app?**

Yes. You should split context by concern and nest the Providers. Components only re-render when the specific context they consume changes.

```jsx
<AuthContext.Provider value={authState}>
  <ThemeContext.Provider value={themeState}>
    <LocaleContext.Provider value={localeState}>
      <App />
    </LocaleContext.Provider>
  </ThemeContext.Provider>
</AuthContext.Provider>

// A component using only ThemeContext is NOT re-rendered when AuthContext changes.
```

---

**7. How do you update context from a child component?**

The Provider passes a setter function as part of the context value. The child calls that setter.

```jsx
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState("light");
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// Inside any deep child:
function ToggleTheme() {
  const { setTheme } = useContext(ThemeContext);
  return <button onClick={() => setTheme("dark")}>Go Dark</button>;
}
```

---

**8. When should you use Context API vs Redux?**

Use Context when the shared data is relatively simple and changes infrequently (auth, theme, locale). Use Redux when the app has complex, frequently changing state, needs middleware for async logic, or requires the Redux DevTools for debugging large state trees.

```
Context API: theme (light/dark), logged-in user, preferred language
Redux:       shopping cart, real-time notifications, complex form wizard state
```

---

**9. What happens if a component calls useContext but there is no Provider?**

It receives the default value passed to `createContext`. There is no error — it silently falls back. This is often the source of subtle bugs when developers forget to add a Provider.

```jsx
const UserContext = createContext(null); // default = null

function UserCard() {
  const user = useContext(UserContext); // null if no Provider above
  if (!user) return <p>No user found</p>; // guard required
}
```

---

**10. What is the custom hook pattern for Context and why use it?**

Wrapping `useContext` in a custom hook provides a cleaner API, allows you to throw an error when the hook is used outside its Provider, and hides the internal implementation from consumers.

```jsx
const AuthContext = createContext(null);

// Custom hook — enforces correct usage
function useAuth() {
  const ctx = useContext(AuthContext);
  if (!ctx) {
    throw new Error("useAuth must be used inside <AuthProvider>");
  }
  return ctx;
}

// Consumer — clean, no direct reference to AuthContext
function Navbar() {
  const { user, logout } = useAuth(); // nice and simple
  return <button onClick={logout}>{user.name}</button>;
}
```
