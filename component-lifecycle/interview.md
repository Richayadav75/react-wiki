# Component Lifecycle Interview Questions

---

**1. What are the three phases of a React component's lifecycle?**

1. **Mount** — the component is created and added to the DOM for the first time.
2. **Update** — the component re-renders because its state or props changed.
3. **Unmount** — the component is removed from the DOM.

```
MOUNT   → useEffect(fn, [])           runs once after first render
UPDATE  → useEffect(fn, [dep])        runs when dep changes
UNMOUNT → useEffect(() => { return cleanup; }, [])  cleanup runs on removal
```

---

**2. How do you replicate componentDidMount in a functional component?**

Use `useEffect` with an empty dependency array. The effect runs once after the first render, just like `componentDidMount`.

```jsx
// Class
componentDidMount() {
  fetchData();
}

// Functional
useEffect(() => {
  fetchData();
}, []); // empty array = runs only on mount
```

---

**3. How do you replicate componentDidUpdate in a functional component?**

Put the value you want to watch in the dependency array. The effect re-runs after mount and whenever that value changes.

```jsx
// Class
componentDidUpdate(prevProps) {
  if (prevProps.userId !== this.props.userId) {
    this.fetchUser(this.props.userId);
  }
}

// Functional
useEffect(() => {
  fetchUser(userId);
}, [userId]); // re-runs on mount AND when userId changes
```

---

**4. How do you replicate componentWillUnmount in a functional component?**

Return a function from `useEffect`. React calls this cleanup function when the component unmounts.

```jsx
// Class
componentWillUnmount() {
  clearInterval(this.timer);
}

// Functional
useEffect(() => {
  const timer = setInterval(tick, 1000);

  return () => clearInterval(timer); // cleanup on unmount
}, []);
```

---

**5. When exactly does the cleanup function run?**

It runs in two situations:
1. When the component unmounts (removed from the DOM).
2. Before the effect runs again (when dependencies change) — to clean up the previous effect.

```jsx
useEffect(() => {
  const sub = subscribe(roomId);

  return () => sub.unsubscribe(); // runs when roomId changes AND on unmount
}, [roomId]);

// Timeline with roomId = "a" → "b":
// 1. Subscribe to "a"
// 2. roomId changes to "b" → unsubscribe from "a" (cleanup)
// 3. Subscribe to "b"
// 4. Unmount → unsubscribe from "b" (cleanup)
```

---

**6. What happens if you omit the dependency array in useEffect?**

The effect runs after every single render. This can cause infinite loops if you update state inside the effect.

```jsx
// Runs after EVERY render
useEffect(() => {
  document.title = count; // fine, no state update
});

// DANGER — infinite loop
useEffect(() => {
  setCount(count + 1); // state update → re-render → effect runs → state update → ...
}); // no dep array
```

---

**7. What is the difference between useEffect and useLayoutEffect?**

- `useEffect` runs *asynchronously* after the browser has painted the screen. Use for most side effects.
- `useLayoutEffect` runs *synchronously* after DOM mutations but *before* the browser paints. Use when you need to read or mutate the DOM before the user sees it (e.g., measuring element dimensions to prevent flicker).

```jsx
useEffect(() => {
  // Fires AFTER paint — user briefly sees the old layout
  setHeight(ref.current.offsetHeight);
});

useLayoutEffect(() => {
  // Fires BEFORE paint — no visible flicker
  setHeight(ref.current.offsetHeight);
});
```

---

**8. What is getDerivedStateFromProps and how is it handled in hooks?**

`getDerivedStateFromProps` is a class method that returns state derived from props before every render. In functional components, you simply compute the value during render — no hook needed for pure derivation.

```jsx
// Class
static getDerivedStateFromProps(props) {
  return { isAdmin: props.role === "admin" };
}

// Functional — just compute inline
function UserBadge({ role }) {
  const isAdmin = role === "admin"; // derived, not stored in state
  return <span>{isAdmin ? "Admin" : "User"}</span>;
}
```

---

**9. What is shouldComponentUpdate and what is its hook-world equivalent?**

`shouldComponentUpdate` returns true/false to tell React whether to re-render. In functional components, `React.memo` wraps the component and performs a shallow prop comparison, skipping re-renders when props haven't changed.

```jsx
// Class
shouldComponentUpdate(nextProps) {
  return nextProps.name !== this.props.name;
}

// Functional
const Card = React.memo(function Card({ name }) {
  return <div>{name}</div>;
});
// Card only re-renders when name prop changes

// Custom comparison:
const Card = React.memo(Card, (prev, next) => prev.id === next.id);
```

---

**10. Show the complete side-by-side lifecycle: class component vs hooks for a data-fetching component.**

```jsx
// CLASS — fetch on mount, re-fetch when id changes, cancel on unmount
class Article extends React.Component {
  state = { article: null, loading: true };

  componentDidMount()  { this.load(this.props.id); }

  componentDidUpdate(prevProps) {
    if (prevProps.id !== this.props.id) {
      this.setState({ loading: true });
      this.load(this.props.id);
    }
  }

  componentWillUnmount() { this.cancelled = true; }

  load(id) {
    fetch(`/api/articles/${id}`)
      .then(r => r.json())
      .then(article => {
        if (!this.cancelled) this.setState({ article, loading: false });
      });
  }

  render() {
    if (this.state.loading) return <p>Loading...</p>;
    return <h1>{this.state.article.title}</h1>;
  }
}

// FUNCTIONAL — same behaviour, one useEffect handles all three phases
function Article({ id }) {
  const [article, setArticle] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    setLoading(true);
    const controller = new AbortController();

    fetch(`/api/articles/${id}`, { signal: controller.signal })
      .then(r => r.json())
      .then(data => { setArticle(data); setLoading(false); })
      .catch(err => { if (err.name !== "AbortError") setLoading(false); });

    return () => controller.abort(); // cleanup = componentWillUnmount
  }, [id]);                          // [id] = componentDidMount + componentDidUpdate

  if (loading) return <p>Loading...</p>;
  return <h1>{article.title}</h1>;
}
```
