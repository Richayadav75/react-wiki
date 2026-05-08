- Category: React Core
- Difficulty: Intermediate
- Related: hooks, useEffect, useState, class-vs-function-component

### Component Lifecycle — Mount, Update, and Unmount

Every React component goes through three phases during its existence: it is born (mounted), it can change (updated), and eventually it is removed (unmounted). Understanding this lifecycle is essential for fetching data, setting up subscriptions, and cleaning up resources correctly.

**Analogy**
A restaurant: mounting is opening day (set up tables, turn on lights). Updating is when customers arrive and orders change (rearrange tables, change the menu). Unmounting is closing night (clean up, turn off lights, lock the door). If you forget to "lock the door" (cleanup), problems accumulate over time.

---

### 1. The Three Lifecycle Phases

**Theory**
Each phase maps to specific class lifecycle methods and their hook equivalents. The underlying concept is the same regardless of whether you use classes or functions.


**Working Flow**
```
Component created
       |
       v
  [ MOUNT ]
  Initial render
  useEffect(fn, []) runs   ← componentDidMount equivalent
       |
       v
  [ UPDATE ]
  State or props change
  Component re-renders
  useEffect(fn, [dep]) runs when dep changes  ← componentDidUpdate equivalent
       |
       | (may update many times)
       v
  [ UNMOUNT ]
  Component removed from DOM
  useEffect cleanup fn runs  ← componentWillUnmount equivalent
```

---

### 2. Mount Phase — componentDidMount → useEffect with []

**Theory**
The mount phase is the ideal time to fetch initial data, set up subscriptions, or start timers. In class components this is `componentDidMount`. With hooks, use `useEffect` with an empty dependency array — it runs exactly once after the first render.

**Working Flow**
![flow-chart-2](flow-chart-2.png)

**Example — Side by Side**
```jsx
// CLASS — mount
class UserList extends React.Component {
  state = { users: [], loading: true };

  componentDidMount() {  // runs once after first render
    fetch("/api/users")
      .then(res => res.json())
      .then(users => this.setState({ users, loading: false }));
  }

  render() {
    if (this.state.loading) return <p>Loading...</p>;
    return <ul>{this.state.users.map(u => <li key={u.id}>{u.name}</li>)}</ul>;
  }
}

// FUNCTIONAL — mount
import { useState, useEffect } from "react";

function UserList() {
  const [users, setUsers]     = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {           // runs once after first render
    fetch("/api/users")
      .then(res => res.json())
      .then(data => {
        setUsers(data);
        setLoading(false);
      });
  }, []);                     // empty array = on mount only

  if (loading) return <p>Loading...</p>;
  return <ul>{users.map(u => <li key={u.id}>{u.name}</li>)}</ul>;
}
```

**Output**
```
Mount:  "Loading..."
After fetch resolves:
  • Alice
  • Bob
  • Charlie
```

---

### 3. Update Phase — componentDidUpdate → useEffect with [deps]

**Theory**
The update phase runs after every state or prop change. In class components, `componentDidUpdate(prevProps, prevState)` lets you compare old and new values. With hooks, you put the value you want to watch in the dependency array — the effect re-runs only when that value changes.

**Working Flow**
![flow-chart-3](flow-chart-3.png)

**Example — Side by Side**
```jsx
// CLASS — re-fetch when userId prop changes
class UserProfile extends React.Component {
  state = { user: null };

  componentDidMount() {
    this.fetchUser(this.props.userId);
  }

  componentDidUpdate(prevProps) {
    if (prevProps.userId !== this.props.userId) { // compare old and new
      this.fetchUser(this.props.userId);
    }
  }

  fetchUser(id) {
    fetch(`/api/users/${id}`)
      .then(res => res.json())
      .then(user => this.setState({ user }));
  }

  render() {
    return <p>{this.state.user?.name ?? "Loading..."}</p>;
  }
}

// FUNCTIONAL — same behaviour, much less code
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {                         // runs on mount AND when userId changes
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(setUser);
  }, [userId]);                             // re-run when userId changes

  return <p>{user?.name ?? "Loading..."}</p>;
}
```

**Output**
```
userId = 1:  "Loading..." → "Alice"
userId = 2:  "Loading..." → "Bob"   (effect ran again because userId changed)
```

---

### 4. Unmount Phase — componentWillUnmount → useEffect cleanup

**Theory**
When a component is removed from the DOM, it must clean up any ongoing work — timers, subscriptions, event listeners. If you don't, these continue running in the background, causing memory leaks or bugs where callbacks fire on unmounted components.

**Working Flow**
![flow-chart-4](flow-chart-4.png)

**Example — Side by Side**
```jsx
// CLASS — cleanup on unmount
class LiveClock extends React.Component {
  state = { time: new Date() };

  componentDidMount() {
    this.timer = setInterval(() => {
      this.setState({ time: new Date() });
    }, 1000);
  }

  componentWillUnmount() {       // cleanup
    clearInterval(this.timer);
  }

  render() {
    return <p>{this.state.time.toLocaleTimeString()}</p>;
  }
}

// FUNCTIONAL — setup + cleanup in one useEffect
function LiveClock() {
  const [time, setTime] = useState(new Date());

  useEffect(() => {
    const timer = setInterval(() => setTime(new Date()), 1000);

    return () => clearInterval(timer); // cleanup when unmounted
  }, []);

  return <p>{time.toLocaleTimeString()}</p>;
}
```

**Output**
```
Component mounted:
  12:00:01 → 12:00:02 → 12:00:03 ...

Component unmounted (e.g., navigated away):
  interval cleared — no more ticks, no memory leak
```

---

### 5. The Cleanup Runs Before the Next Effect Too

**Theory**
The cleanup function does not only run on unmount — it also runs *before* the effect fires again. This ensures each effect starts with a clean slate. This is a subtle but important difference from `componentWillUnmount`.

**Working Flow**
![flow-chart-5](flow-chart-5.png)

**Example**
```jsx
function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();
    console.log(`Connected to room ${roomId}`);

    return () => {
      connection.disconnect();
      console.log(`Disconnected from room ${roomId}`);
    };
  }, [roomId]); // re-run when roomId changes

  return <p>You are in room {roomId}</p>;
}
```

**Output**
```
roomId = "general":
  "Connected to room general"

roomId changes to "react":
  "Disconnected from room general"   ← cleanup ran first
  "Connected to room react"

Component unmounts:
  "Disconnected from room react"
```

---

### 6. getDerivedStateFromProps — Hook Equivalent

**Theory**
`getDerivedStateFromProps` is a class lifecycle method that runs before every render, returning state derived from props. In functional components, you simply derive the value during render — no side effect or hook needed.

**Working Flow**
![flow-chart-6](flow-chart-6.png)

**Example**
```jsx
// CLASS approach
class UserBadge extends React.Component {
  static getDerivedStateFromProps(props) {
    return { isAdmin: props.role === "admin" };
  }

  render() {
    return <span>{this.state.isAdmin ? "Admin" : "User"}</span>;
  }
}

// FUNCTIONAL — no hook needed, derive inline
function UserBadge({ role }) {
  const isAdmin = role === "admin"; // computed during render
  return <span>{isAdmin ? "Admin" : "User"}</span>;
}
```

**Output**
```
role="admin"  → "Admin"
role="viewer" → "User"
```

---

### 7. shouldComponentUpdate — React.memo Equivalent

**Theory**
`shouldComponentUpdate` lets you skip a re-render when props/state haven't meaningfully changed. In functional components, `React.memo` provides the same optimisation — it does a shallow comparison of props and skips re-rendering if nothing changed.

**Working Flow**
![flow-chart-7](flow-chart-7.png)

**Example**
```jsx
// Every render of App used to re-render ExpensiveCard even if name didn't change
function ExpensiveCard({ name }) {
  console.log("Card rendered:", name);
  return <div className="expensive">{name}</div>;
}

// Wrap with React.memo — now only re-renders when name prop changes
const OptimisedCard = React.memo(ExpensiveCard);

function App() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>
      <OptimisedCard name="Alice" />   {/* does NOT re-render on count change */}
    </div>
  );
}
```

**Output**
```
Mount:         console: "Card rendered: Alice"
Click +1:      console: nothing  (Card skipped re-render — name unchanged)
Click +1 again: console: nothing  (same)
```

---

### Real-World Example — Fetch on Mount, Cleanup on Unmount

```jsx
import { useState, useEffect } from "react";

function ArticleDetail({ articleId }) {
  const [article, setArticle]   = useState(null);
  const [loading, setLoading]   = useState(true);
  const [error, setError]       = useState(null);

  useEffect(() => {
    // Reset state when articleId changes (update phase)
    setLoading(true);
    setArticle(null);
    setError(null);

    const controller = new AbortController();

    fetch(`/api/articles/${articleId}`, { signal: controller.signal })
      .then(res => {
        if (!res.ok) throw new Error("Not found");
        return res.json();
      })
      .then(data => {
        setArticle(data);
        setLoading(false);
      })
      .catch(err => {
        if (err.name !== "AbortError") { // ignore intentional cancellation
          setError(err.message);
          setLoading(false);
        }
      });

    // Cleanup: cancel in-flight request when articleId changes or component unmounts
    return () => controller.abort();
  }, [articleId]); // re-run on mount + whenever articleId changes

  if (loading) return <p>Loading article {articleId}...</p>;
  if (error)   return <p style={{ color: "red" }}>Error: {error}</p>;

  return (
    <article>
      <h1>{article.title}</h1>
      <p>{article.body}</p>
    </article>
  );
}
```

**Output**
```
Mount with articleId=1:
  "Loading article 1..."  → renders article title and body

articleId changes to 2:
  "Loading article 2..."  → (request for article 1 is aborted)
  → renders article 2 content

Component unmounts (navigate away):
  → current request aborted (no stale setState call)

Lifecycle summary:
  Mount    → fetch begins    (useEffect with [articleId] on first render)
  Update   → fetch re-runs   (when articleId prop changes)
  Unmount  → fetch cancelled (cleanup function runs → controller.abort())
```

---

[View Interview Questions](./interview.md)
