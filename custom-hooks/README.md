- Category: React Patterns
- Difficulty: Intermediate
- Related: useState, useEffect, useRef, closures

### Custom Hooks — Extracting and Reusing Stateful Logic
A **Custom Hook** is a JavaScript function whose name starts with `use` and that calls other React Hooks inside it. Think of it as a way to package a piece of stateful behavior — like fetching data or listening to window size — so any component can plug into it without rewriting the logic.

**Analogy**
A custom hook is like a power strip. Your components are devices (laptop, lamp, phone). Instead of hard-wiring every device to the wall socket, you plug once into the strip. Each device gets its own independent current — they don't share it. Custom hooks work the same way: each component that uses the hook gets its own isolated state.

---

### 1. The Naming Rule — Why "use" Matters

**Theory**: React's linter (ESLint plugin for hooks) uses the `use` prefix as a signal that a function follows the Rules of Hooks. If your function calls `useState` or `useEffect` but is NOT named with `use`, React cannot warn you when you break the rules (calling inside loops, conditionals, etc.).

**Working Flow**
![flow-chart](flow-chart.png)

**Rules of Custom Hooks**
- Name must start with `use` (e.g., `useFetch`, `useToggle`, `useDebounce`)
- Can call other hooks (`useState`, `useEffect`, `useRef`, other custom hooks)
- Cannot be called inside loops, conditions, or nested functions
- Returns data or functions — never JSX

**Example**
```jsx
// WRONG — not named with "use", linter cannot protect you
function fetchData(url) {
  const [data, setData] = useState(null); // will fail hook rules silently
  return data;
}

// CORRECT
function useFetchData(url) {
  const [data, setData] = useState(null);
  useEffect(() => {
    fetch(url).then(r => r.json()).then(setData);
  }, [url]);
  return data;
}
```

**Output**
```
useFetchData("/api/users") → returns: null (initially), then [{ id:1, name:"Alice" }, ...]
fetchData("/api/users")    → React throws: Invalid hook call error
```

---

### 2. Before vs After — Extracting Logic from a Component

**Theory**: Without custom hooks, stateful logic lives directly inside components. This makes it impossible to reuse and hard to test. Extracting to a custom hook separates concerns cleanly.

**Working Flow**
![flow-chart-2](flow-chart-2.png)

**Before — logic inside component**
```jsx
// Every component that fetches data repeats this block
function UserProfile({ userId }) {
  const [data, setData]       = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError]     = useState(null);

  useEffect(() => {
    setLoading(true);
    fetch(`/api/users/${userId}`)
      .then(r => {
        if (!r.ok) throw new Error("Network error");
        return r.json();
      })
      .then(d => { setData(d); setLoading(false); })
      .catch(e => { setError(e.message); setLoading(false); });
  }, [userId]);

  if (loading) return <p>Loading...</p>;
  if (error)   return <p>Error: {error}</p>;
  return <h1>{data.name}</h1>;
}
```

**After — logic in useFetch**
```jsx
// hooks/useFetch.js
function useFetch(url) {
  const [data, setData]       = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError]     = useState(null);

  useEffect(() => {
    let cancelled = false;
    setLoading(true);
    fetch(url)
      .then(r => {
        if (!r.ok) throw new Error(`HTTP ${r.status}`);
        return r.json();
      })
      .then(d  => { if (!cancelled) { setData(d);         setLoading(false); } })
      .catch(e => { if (!cancelled) { setError(e.message); setLoading(false); } });
    return () => { cancelled = true; };  // cleanup on unmount / url change
  }, [url]);

  return { data, loading, error };
}

// UserProfile — now clean
function UserProfile({ userId }) {
  const { data, loading, error } = useFetch(`/api/users/${userId}`);

  if (loading) return <p>Loading...</p>;
  if (error)   return <p>Error: {error}</p>;
  return <h1>{data.name}</h1>;
}

// ProductPage — reuses exact same hook, own independent state
function ProductPage({ productId }) {
  const { data, loading, error } = useFetch(`/api/products/${productId}`);

  if (loading) return <p>Loading product...</p>;
  if (error)   return <p>Failed: {error}</p>;
  return <h2>{data.title} — ${data.price}</h2>;
}
```

**Output**
```
UserProfile (userId=1):
  Loading...          ← while fetching
  Alice Johnson       ← after fetch resolves

ProductPage (productId=42):
  Loading product...  ← while fetching (independent state from UserProfile)
  MacBook Pro — $1999 ← after fetch resolves
```

---

### 3. useLocalStorage — Syncing State to the Browser

**Theory**: `localStorage` persists data between page reloads, but React's `useState` resets on mount. A `useLocalStorage` hook combines both — state is initialized from storage and every update is written back automatically.

**Working Flow**
![flow-chart-3](flow-chart-3.png)

**Example**
```jsx
function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });

  const setValue = (value) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(error);
    }
  };

  return [storedValue, setValue];
}

// Usage
function ThemeToggle() {
  const [theme, setTheme] = useLocalStorage("theme", "light");

  return (
    <div>
      <p>Current theme: {theme}</p>
      <button onClick={() => setTheme(t => t === "light" ? "dark" : "light")}>
        Toggle Theme
      </button>
    </div>
  );
}
```

**Output**
```
First visit:
  Current theme: light
  [Toggle Theme button]

After clicking toggle:
  Current theme: dark

After page reload:
  Current theme: dark   ← persisted from localStorage
```

---

### 4. useDebounce — Delay Expensive Operations

**Theory**: Debouncing means waiting until the user stops doing something (typing, resizing) before triggering an expensive action (API call, heavy computation). Without debounce, every keystroke fires a network request.

**Working Flow**
![flow-chart-4](flow-chart-4.png)

**Example**
```jsx
function useDebounce(value, delay = 500) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => clearTimeout(timer);  // cancel if value changes again
  }, [value, delay]);

  return debouncedValue;
}

// Usage — search component
function SearchBar() {
  const [query, setQuery]   = useState("");
  const debouncedQuery      = useDebounce(query, 500);
  const { data: results }   = useFetch(
    debouncedQuery ? `/api/search?q=${debouncedQuery}` : null
  );

  return (
    <div>
      <input
        value={query}
        onChange={e => setQuery(e.target.value)}
        placeholder="Search..."
      />
      <ul>
        {results?.map(r => <li key={r.id}>{r.name}</li>)}
      </ul>
    </div>
  );
}
```

**Output**
```
User types "react" quickly (5 keystrokes in 400ms):
  query state    → "r", "re", "rea", "reac", "react"   (updates every keystroke)
  debouncedQuery → "react"                              (only after 500ms silence)
  API calls made → 1  (not 5)
```

---

### 5. useWindowSize — Responsive Logic in JS

**Theory**: Sometimes CSS media queries aren't enough — you need JS to know the window size (e.g., conditionally rendering a mobile menu). This hook listens to `resize` events and always returns the current dimensions.

**Working Flow**
![flow-chart-5](flow-chart-5.png)

**Example**
```jsx
function useWindowSize() {
  const [size, setSize] = useState({
    width:  window.innerWidth,
    height: window.innerHeight,
  });

  useEffect(() => {
    function handleResize() {
      setSize({ width: window.innerWidth, height: window.innerHeight });
    }
    window.addEventListener("resize", handleResize);
    return () => window.removeEventListener("resize", handleResize);
  }, []);  // [] = only attach once on mount

  return size;
}

// Usage
function Layout() {
  const { width } = useWindowSize();
  const isMobile  = width < 768;

  return (
    <div>
      {isMobile ? <MobileMenu /> : <DesktopNav />}
      <main>Content here</main>
    </div>
  );
}
```

**Output**
```
Width: 1200px → renders: <DesktopNav />
Width:  600px → renders: <MobileMenu />
Width:  400px → renders: <MobileMenu />
```

---

### 6. useOnClickOutside — Dismiss Dropdowns and Modals

**Theory**: A very common UI pattern — clicking outside a dropdown/modal closes it. Instead of adding click listeners in every dropdown component, extract the logic once.

**Working Flow**
![flow-chart-6](flow-chart-6.png)

**Example**
```jsx
function useOnClickOutside(ref, handler) {
  useEffect(() => {
    function listener(event) {
      if (!ref.current || ref.current.contains(event.target)) return;
      handler(event);
    }
    document.addEventListener("mousedown", listener);
    document.addEventListener("touchstart", listener);
    return () => {
      document.removeEventListener("mousedown", listener);
      document.removeEventListener("touchstart", listener);
    };
  }, [ref, handler]);
}

// Usage
function Dropdown() {
  const [open, setOpen] = useState(false);
  const ref = useRef(null);

  useOnClickOutside(ref, () => setOpen(false));

  return (
    <div ref={ref}>
      <button onClick={() => setOpen(o => !o)}>Menu</button>
      {open && (
        <ul>
          <li>Profile</li>
          <li>Settings</li>
          <li>Logout</li>
        </ul>
      )}
    </div>
  );
}
```

**Output**
```
Click "Menu" button  → dropdown opens (open: true)
Click inside list    → dropdown stays open
Click outside div    → dropdown closes (open: false)
```

---

### Real-World Examples

**E-commerce site**
```text
useCart()          → add/remove items, total, item count
useWishlist()      → toggle saved items
useProductFilter() → filter by price/category with debounce
useInventory(id)   → real-time stock status via polling
```

**Dashboard app**
```text
useAuth()          → current user, login, logout
usePermissions()   → role-based access check
useWebSocket(url)  → live data streaming
usePagination()    → page, pageSize, goToPage, totalPages
```

**Composing hooks**
```jsx
// Hooks can call other custom hooks
function useAuthenticatedFetch(endpoint) {
  const { user }              = useAuth();             // custom hook
  const { data, loading }     = useFetch(
    user ? `/api${endpoint}` : null                    // custom hook
  );
  return { data, loading, isLoggedIn: !!user };
}
```

---

[View Interview Questions](./interview.md)
