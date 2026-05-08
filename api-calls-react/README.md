- Category: Data Fetching
- Difficulty: Intermediate
- Related: useEffect, promises, async-await

### API Calls in React
React is a UI library — it doesn't know how to fetch data on its own. You wire it up with the browser's built-in `fetch()` or a library like Axios inside a `useEffect` hook, then store the result in state so the component can re-render with real data.

**Analogy**
A restaurant waiter. You (component) sit down and place an order (useEffect runs). The waiter (fetch) goes to the kitchen (API server). While waiting you see a "Loading..." message. The waiter returns with food (data) or says "Sorry, we're out" (error). The table is then updated (setState → re-render).

---

### 1. The Loading / Error / Data State Pattern
**Theory**: Every API call has three possible states — loading, error, and success. You track all three with `useState`. Never skip the loading or error state — your users will thank you.

**Working Flow**
![flow-chart](flow-chart.png)

**Example**
```jsx
import { useState, useEffect } from "react";

function PostsList() {
  const [posts, setPosts]     = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError]     = useState(null);

  useEffect(() => {
    fetch("https://jsonplaceholder.typicode.com/posts")
      .then(res => {
        if (!res.ok) throw new Error(`HTTP error ${res.status}`);
        return res.json();
      })
      .then(data => {
        setPosts(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, []); // empty [] = run once on mount

  if (loading) return <p>Loading posts...</p>;
  if (error)   return <p style={{ color: "red" }}>Error: {error}</p>;

  return (
    <ul>
      {posts.slice(0, 3).map(p => (
        <li key={p.id}>{p.title}</li>
      ))}
    </ul>
  );
}
```

**Output**
```
(on mount)  → "Loading posts..."
(success)   → <ul>
                <li>sunt aut facere...</li>
                <li>qui est esse</li>
                <li>ea molestias quasi...</li>
              </ul>
(on error)  → "Error: Failed to fetch"
```

**Explanation**: `useEffect` with `[]` runs exactly once after the first render. The three-state pattern (loading / error / data) is the industry standard — always initialize `loading` to `true` so the spinner shows immediately.

---

### 2. Async/Await with Cleanup (AbortController)
**Theory**: Using `async/await` inside `useEffect` is cleaner. But there is a subtle bug — if the component unmounts while the request is in flight, React will try to call `setState` on an unmounted component. The fix is `AbortController`, which lets you cancel the fetch.

**Working Flow**
![flow-chart-2](flow-chart-2.png)

**Example**
```jsx
useEffect(() => {
  const controller = new AbortController();

  async function loadPosts() {
    try {
      setLoading(true);
      const res  = await fetch("https://jsonplaceholder.typicode.com/posts", {
        signal: controller.signal,   // attach cancellation signal
      });
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      const data = await res.json();
      setPosts(data);
    } catch (err) {
      if (err.name === "AbortError") return; // ignore cancellation
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }

  loadPosts();

  return () => controller.abort(); // cleanup on unmount
}, []);
```

**Output**
```
Normal flow  → posts array set, loading=false
Unmount fast → AbortError caught → ignored safely → no setState warning
HTTP 404     → "Error: HTTP 404"
```

**Explanation**: The cleanup function returned from `useEffect` runs when the component unmounts or before the effect re-runs. Calling `controller.abort()` sends a signal to the in-flight request — the `catch` block catches it as `AbortError` and returns early.

---

### 3. Race Conditions and the Fix
**Theory**: A race condition happens when a user changes a filter quickly (e.g., tabs from "Sports" to "Tech"). Two fetches are now in flight. If the "Sports" response arrives after "Tech", your UI shows stale data. AbortController is the fix.

**Working Flow**
![flow-chart-3](flow-chart-3.png)

**Example**
```jsx
function NewsFeed({ category }) {
  const [articles, setArticles] = useState([]);

  useEffect(() => {
    const controller = new AbortController();

    async function load() {
      try {
        const res  = await fetch(`/api/news?category=${category}`, {
          signal: controller.signal,
        });
        const data = await res.json();
        setArticles(data);
      } catch (err) {
        if (err.name !== "AbortError") console.error(err);
      }
    }

    load();
    return () => controller.abort(); // aborts previous fetch when category changes
  }, [category]); // re-runs every time category changes

  return <ul>{articles.map(a => <li key={a.id}>{a.title}</li>)}</ul>;
}
```

**Output**
```
category="Sports" → fetch starts [A]
category="Tech"   → [A] aborted, fetch starts [B]
[B] resolves      → articles = techArticles  ← always correct
```

---

### 4. Axios vs Fetch
**Theory**: `fetch` is built-in to the browser. `axios` is a third-party library (~15kB) that adds conveniences: automatic JSON parsing, request/response interceptors, and it throws on 4xx/5xx status codes (unlike fetch which only throws on network failure).

**Working Flow**
![flow-chart-4](flow-chart-4.png)

**Example**
```jsx
// ---- fetch (manual checks needed) ----
const res = await fetch("/api/users");
if (!res.ok) throw new Error(`HTTP ${res.status}`);  // must check!
const data = await res.json();                        // must parse!

// ---- axios (automatic) ----
import axios from "axios";
const { data } = await axios.get("/api/users");       // parsed + throws on error

// axios with config
const { data: user } = await axios.post("/api/users", {
  name: "Richa",
  role: "admin",
}, {
  headers: { Authorization: `Bearer ${token}` },
});
```

**Output**
```
fetch 404    → res.ok = false, must throw manually
axios 404    → throws AxiosError automatically
fetch 200    → need await res.json()
axios 200    → data is already the JS object
```

| Feature | fetch | axios |
| :--- | :--- | :--- |
| Built-in | Yes | No (npm install) |
| Auto JSON parse | No | Yes |
| Throws on 4xx/5xx | No | Yes |
| Interceptors | No | Yes |
| Request cancellation | AbortController | CancelToken / AbortController |

---

### 5. Custom `useFetch` Hook
**Theory**: If you find yourself copy-pasting the same loading/error/data pattern across components, extract it into a custom hook. A custom hook is just a JavaScript function whose name starts with `use` and that can call other hooks.

**Working Flow**
![flow-chart-5](flow-chart-5.png)

**Example**
```jsx
// hooks/useFetch.js
import { useState, useEffect } from "react";

export function useFetch(url) {
  const [data, setData]       = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError]     = useState(null);

  useEffect(() => {
    if (!url) return;
    const controller = new AbortController();

    async function load() {
      setLoading(true);
      setError(null);
      try {
        const res  = await fetch(url, { signal: controller.signal });
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        const json = await res.json();
        setData(json);
      } catch (err) {
        if (err.name !== "AbortError") setError(err.message);
      } finally {
        setLoading(false);
      }
    }

    load();
    return () => controller.abort();
  }, [url]);

  return { data, loading, error };
}

// Usage in a component — zero boilerplate!
function UserProfile({ userId }) {
  const { data: user, loading, error } = useFetch(
    `https://jsonplaceholder.typicode.com/users/${userId}`
  );

  if (loading) return <p>Loading...</p>;
  if (error)   return <p>Error: {error}</p>;
  return <h2>{user.name} — {user.email}</h2>;
}
```

**Output**
```
loading=true  → "Loading..."
loading=false → "Leanne Graham — Sincere@april.biz"
error         → "Error: HTTP 404"
```

---

### 6. React Query (TanStack Query) — The Modern Solution
**Theory**: React Query is a server-state library. It handles caching, background refetching, stale-while-revalidate, pagination, and optimistic updates — all the hard stuff. `useQuery` replaces the entire loading/error/data pattern. `useMutation` handles POST/PUT/DELETE with automatic cache invalidation.

**Working Flow**
![flow-chart-6](flow-chart-6.png)

**Example**
```jsx
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";

const fetchPosts = () =>
  fetch("https://jsonplaceholder.typicode.com/posts").then(r => r.json());

// Reading data
function PostsList() {
  const { data: posts, isLoading, isError, error } = useQuery({
    queryKey: ["posts"],
    queryFn: fetchPosts,
    staleTime: 60_000,   // consider data fresh for 1 minute
  });

  if (isLoading) return <p>Loading...</p>;
  if (isError)   return <p>Error: {error.message}</p>;

  return <ul>{posts.slice(0, 3).map(p => <li key={p.id}>{p.title}</li>)}</ul>;
}

// Writing data (mutation)
function AddPost() {
  const queryClient = useQueryClient();

  const { mutate, isLoading } = useMutation({
    mutationFn: (newPost) =>
      fetch("/api/posts", {
        method: "POST",
        body: JSON.stringify(newPost),
        headers: { "Content-Type": "application/json" },
      }).then(r => r.json()),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["posts"] }); // refetch list
    },
  });

  return (
    <button
      disabled={isLoading}
      onClick={() => mutate({ title: "New Post", body: "Hello!" })}
    >
      {isLoading ? "Saving..." : "Add Post"}
    </button>
  );
}
```

**Output**
```
First visit   → "Loading..." → posts rendered → cached
Second visit  → posts shown INSTANTLY from cache → background refetch
Add Post btn  → "Saving..." → post created → ["posts"] cache invalidated → list refreshes
```

**Explanation**: The `queryKey` array is the cache key. Whenever you call `invalidateQueries` with the same key, React Query marks the cache as stale and re-fetches in the background. This keeps your UI always fresh without manual state management.

---

### Real-World Example: Posts Feed with Loading Spinner and Error
```jsx
import { useState, useEffect } from "react";

function Spinner() {
  return (
    <div style={{ textAlign: "center", padding: "2rem" }}>
      ⏳ Fetching posts...
    </div>
  );
}

function ErrorBanner({ message }) {
  return (
    <div style={{ color: "red", border: "1px solid red", padding: "1rem" }}>
      Something went wrong: {message}
      <br />
      <button onClick={() => window.location.reload()}>Retry</button>
    </div>
  );
}

function PostsFeed() {
  const [posts, setPosts]     = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError]     = useState(null);

  useEffect(() => {
    const controller = new AbortController();
    (async () => {
      try {
        const res  = await fetch(
          "https://jsonplaceholder.typicode.com/posts?_limit=5",
          { signal: controller.signal }
        );
        if (!res.ok) throw new Error(`Server error: ${res.status}`);
        setPosts(await res.json());
      } catch (err) {
        if (err.name !== "AbortError") setError(err.message);
      } finally {
        setLoading(false);
      }
    })();
    return () => controller.abort();
  }, []);

  if (loading) return <Spinner />;
  if (error)   return <ErrorBanner message={error} />;

  return (
    <div>
      <h2>Latest Posts</h2>
      {posts.map(post => (
        <article key={post.id} style={{ marginBottom: "1rem" }}>
          <h3>{post.title}</h3>
          <p>{post.body}</p>
        </article>
      ))}
    </div>
  );
}
```

**Output**
```
(loading)  →  ⏳ Fetching posts...
(success)  →  Latest Posts
              [title 1] [body 1]
              [title 2] [body 2]
              ... (5 posts)
(error)    →  Something went wrong: Server error: 500
              [Retry button]
```

---

[View Interview Questions](./interview.md)
