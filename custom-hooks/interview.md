# Custom Hooks — Interview Questions

---

**1. What is a custom hook and what makes a function a custom hook?**

A custom hook is a regular JavaScript function that starts with `use` and calls other React hooks inside it. The `use` prefix is what makes it a custom hook — React's linter uses it to enforce the Rules of Hooks and warn when hooks are called conditionally or inside loops.

```jsx
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(navigator.onLine);

  useEffect(() => {
    const on  = () => setIsOnline(true);
    const off = () => setIsOnline(false);
    window.addEventListener("online",  on);
    window.addEventListener("offline", off);
    return () => {
      window.removeEventListener("online",  on);
      window.removeEventListener("offline", off);
    };
  }, []);

  return isOnline;
}

function StatusBar() {
  const isOnline = useOnlineStatus();
  return <p>{isOnline ? "Connected" : "Offline"}</p>;
}
```

---

**2. Do two components using the same custom hook share state?**

No. Each component gets its own completely isolated state. The hook is just a pattern — calling it creates a fresh set of state variables for each component instance.

```jsx
function useCounter(start = 0) {
  const [count, setCount] = useState(start);
  return { count, increment: () => setCount(c => c + 1) };
}

function CounterA() {
  const { count, increment } = useCounter(0);  // own state, starts at 0
  return <button onClick={increment}>A: {count}</button>;
}

function CounterB() {
  const { count, increment } = useCounter(10); // own state, starts at 10
  return <button onClick={increment}>B: {count}</button>;
}
// Clicking A's button does NOT affect B's count.
```

---

**3. What problem does a custom hook solve compared to a regular utility function?**

A regular utility function cannot call React hooks. Custom hooks can hold state, run side effects, and clean up after themselves — all tied to the component lifecycle.

```jsx
// Plain utility — no state, no lifecycle
function formatPrice(amount) {
  return `$${amount.toFixed(2)}`;
}

// Custom hook — has state and lifecycle
function useExchangeRate(currency) {
  const [rate, setRate] = useState(1);
  useEffect(() => {
    fetch(`/api/rates/${currency}`)
      .then(r => r.json())
      .then(d => setRate(d.rate));
  }, [currency]);
  return rate;
}
```

---

**4. How do you implement a useFetch hook that avoids memory leaks?**

Use a `cancelled` flag inside the effect. When the component unmounts or the URL changes, the cleanup function sets `cancelled = true`. Async callbacks check this flag before calling setState.

```jsx
function useFetch(url) {
  const [data,    setData]    = useState(null);
  const [loading, setLoading] = useState(true);
  const [error,   setError]   = useState(null);

  useEffect(() => {
    if (!url) return;
    let cancelled = false;
    setLoading(true);

    fetch(url)
      .then(r => r.json())
      .then(d  => { if (!cancelled) { setData(d);          setLoading(false); } })
      .catch(e => { if (!cancelled) { setError(e.message); setLoading(false); } });

    return () => { cancelled = true; };
  }, [url]);

  return { data, loading, error };
}
```

Without `cancelled`, calling `setData` after unmount causes a warning and potential memory leak.

---

**5. What is the difference between a custom hook and a Higher-Order Component (HOC)?**

| Aspect | Custom Hook | HOC |
|---|---|---|
| Syntax | Function returning data | Function wrapping a component |
| Composition | Easy — call multiple hooks | Wrapper hell with multiple HOCs |
| Props collision | Not possible | Possible (name clashes) |
| Debugging | Hook name shows in DevTools | Wrapped component name is hidden |
| Usage era | Modern React (16.8+) | Older pattern, still valid |

```jsx
// HOC approach (older pattern)
const withAuth = (Component) => (props) => {
  const user = getUser();
  return user ? <Component {...props} user={user} /> : <Redirect to="/login" />;
};

// Custom hook approach (modern)
function useAuth() {
  const [user, setUser] = useState(null);
  useEffect(() => { setUser(getUser()); }, []);
  return user;
}
```

---

**6. How does useDebounce work and when would you use it?**

`useDebounce` delays updating a value until the input stops changing for a specified duration. The `useEffect` clears the timer on every change, only letting it fire after the delay.

```jsx
function useDebounce(value, delay = 500) {
  const [debounced, setDebounced] = useState(value);

  useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(id); // reset timer on each change
  }, [value, delay]);

  return debounced;
}

function ProductSearch() {
  const [input, setInput] = useState("");
  const debouncedInput    = useDebounce(input, 400);
  const { data }          = useFetch(
    debouncedInput ? `/api/products?q=${debouncedInput}` : null
  );

  return (
    <>
      <input value={input} onChange={e => setInput(e.target.value)} />
      {data?.map(p => <p key={p.id}>{p.name}</p>)}
    </>
  );
}
// User types "react" in 5 keystrokes → only 1 API call fires (after 400ms pause)
```

---

**7. How do you persist state across page reloads using a custom hook?**

Use `useLocalStorage`. Initialize state by reading from `localStorage` (lazy initializer), and write back on every state update.

```jsx
function useLocalStorage(key, defaultValue) {
  const [value, setValue] = useState(() => {
    try {
      const saved = localStorage.getItem(key);
      return saved !== null ? JSON.parse(saved) : defaultValue;
    } catch {
      return defaultValue;
    }
  });

  const set = (newValue) => {
    const next = typeof newValue === "function" ? newValue(value) : newValue;
    setValue(next);
    localStorage.setItem(key, JSON.stringify(next));
  };

  return [value, set];
}

function Settings() {
  const [lang, setLang] = useLocalStorage("lang", "en");
  return (
    <select value={lang} onChange={e => setLang(e.target.value)}>
      <option value="en">English</option>
      <option value="fr">French</option>
    </select>
  );
}
// After page reload → lang still reads from localStorage, not reset to "en"
```

---

**8. Can custom hooks be composed? Give an example.**

Yes. Custom hooks can call other custom hooks, allowing complex behavior to be built from smaller pieces.

```jsx
function useLocalStorage(key, initial) { /* ... */ }
function useFetch(url)                 { /* ... */ }

// Composed hook — uses both primitives
function useUserPreferences(userId) {
  const [prefs, setPrefs]     = useLocalStorage(`prefs-${userId}`, {});
  const { data: serverPrefs } = useFetch(`/api/users/${userId}/preferences`);

  const merged = { ...serverPrefs, ...prefs }; // local overrides server

  return {
    preferences: merged,
    updatePreference: (k, v) => setPrefs(p => ({ ...p, [k]: v })),
  };
}
```

---

**9. What does "stale closure" mean in a custom hook and how do you fix it?**

A stale closure happens when an effect's callback captures an old value of a variable and never updates. The fix is to store the latest callback in a `useRef`.

```jsx
// Bug: setInterval always calls the initial version of callback
function useInterval(callback, delay) {
  useEffect(() => {
    const id = setInterval(callback, delay); // stale closure — callback never updates
    return () => clearInterval(id);
  }, [delay]); // callback missing from deps
}

// Fix: keep latest callback in a ref
function useInterval(callback, delay) {
  const savedCallback = useRef(callback);

  useEffect(() => { savedCallback.current = callback; }, [callback]);

  useEffect(() => {
    const id = setInterval(() => savedCallback.current(), delay);
    return () => clearInterval(id);
  }, [delay]);
}
```

---

**10. When should you NOT create a custom hook?**

- When the logic is only used in one place and is unlikely to be reused.
- When it has no state or side effects — a plain function is simpler.
- When the abstraction adds more confusion than clarity.

```jsx
// Over-engineered (just use useState directly)
function useVisible() {
  const [v, setV] = useState(true);
  return [v, () => setV(x => !x)];
}

// Better — inline it
const [visible, setVisible] = useState(true);
```

Create a custom hook when: the logic spans multiple hooks, it is used in more than one component, or it manages a complex side effect like subscriptions, timers, or fetch with cleanup.
