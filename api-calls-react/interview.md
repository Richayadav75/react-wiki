# API Calls in React — Interview Questions

---

**1. Where should you make an API call in a React functional component, and why?**

Inside the `useEffect` hook. React renders the UI first, then fires effects. If you called `fetch` at the top level of the component body, it would run on every render (including renders triggered by the state update from the fetch itself), causing infinite loops.

```jsx
useEffect(() => {
  fetch("/api/data").then(r => r.json()).then(setData);
}, []); // [] = run once after first render
```

---

**2. What is the three-state pattern for API calls?**

Every fetch has three states — always track all three:

```jsx
const [data, setData]       = useState(null);   // server response
const [loading, setLoading] = useState(true);   // show spinner
const [error, setError]     = useState(null);   // show error message

// In useEffect:
try {
  const res  = await fetch(url);
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  setData(await res.json());
} catch (err) {
  setError(err.message);
} finally {
  setLoading(false); // always stop loading
}
```

Skipping the error state means failed requests show a blank screen instead of a helpful message.

---

**3. What is a race condition in data fetching, and how do you fix it?**

A race condition occurs when two fetches are in flight at the same time (e.g., user changes a filter twice quickly). The older request may arrive after the newer one, overwriting the correct data.

Fix with `AbortController`:

```jsx
useEffect(() => {
  const controller = new AbortController();

  fetch(`/api/posts?category=${category}`, { signal: controller.signal })
    .then(r => r.json())
    .then(setPosts)
    .catch(err => {
      if (err.name !== "AbortError") setError(err.message);
    });

  return () => controller.abort(); // cancel previous request when category changes
}, [category]);
```

When `category` changes, React runs the cleanup function (which aborts the old fetch) before starting the new effect.

---

**4. Why can't you make the `useEffect` callback itself `async`?**

`async` functions return a Promise. `useEffect` expects its callback to return either nothing or a cleanup function. Returning a Promise breaks React's cleanup mechanism.

```jsx
// WRONG — useEffect gets a Promise, not a cleanup function
useEffect(async () => {
  const data = await fetch(url).then(r => r.json());
  setData(data);
}, []);

// CORRECT — define async function inside, call it immediately
useEffect(() => {
  async function load() {
    const data = await fetch(url).then(r => r.json());
    setData(data);
  }
  load();
}, []);
```

---

**5. What are the key differences between `fetch` and `axios`?**

| Aspect | fetch | axios |
| :--- | :--- | :--- |
| Built-in | Yes (no install) | npm install axios |
| JSON parsing | Manual (`await res.json()`) | Automatic (`res.data`) |
| 4xx/5xx errors | Does NOT throw — need `res.ok` check | Throws automatically |
| Request interceptors | No | Yes |
| Upload progress | No | Yes |

```jsx
// fetch — manual error check required
const res = await fetch("/api/users");
if (!res.ok) throw new Error("Failed");   // without this, 404 silently succeeds
const data = await res.json();

// axios — cleaner
const { data } = await axios.get("/api/users"); // throws on 4xx/5xx automatically
```

---

**6. What is a custom `useFetch` hook and when would you build one?**

When multiple components share the same fetch + loading + error logic, extract it into a custom hook to avoid duplication:

```jsx
function useFetch(url) {
  const [data, setData]       = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError]     = useState(null);

  useEffect(() => {
    const controller = new AbortController();
    setLoading(true);
    fetch(url, { signal: controller.signal })
      .then(r => r.json())
      .then(setData)
      .catch(err => { if (err.name !== "AbortError") setError(err.message); })
      .finally(() => setLoading(false));
    return () => controller.abort();
  }, [url]);

  return { data, loading, error };
}

// Usage
const { data: user, loading } = useFetch(`/api/users/${id}`);
```

---

**7. What is React Query (TanStack Query) and what problems does it solve?**

React Query is a server-state library. It solves caching, deduplication, background refetching, stale data, and optimistic updates — problems that `useState + useEffect` cannot solve elegantly.

```jsx
// Without React Query: ~30 lines of useState/useEffect boilerplate
// With React Query:
const { data, isLoading, isError } = useQuery({
  queryKey: ["posts"],
  queryFn: () => fetch("/api/posts").then(r => r.json()),
  staleTime: 60_000, // serve from cache for 1 min
});
```

Key advantages:
- Automatic caching keyed by `queryKey`
- Background refetch when tab regains focus
- `useMutation` + `invalidateQueries` for write + cache sync
- No race conditions by design

---

**8. How does `useMutation` with optimistic updates work?**

An optimistic update immediately shows the expected result before the server confirms, giving instant feedback. If the server fails, you roll back.

```jsx
const queryClient = useQueryClient();

const { mutate } = useMutation({
  mutationFn: (newPost) =>
    fetch("/api/posts", { method: "POST", body: JSON.stringify(newPost) }),

  onMutate: async (newPost) => {
    await queryClient.cancelQueries({ queryKey: ["posts"] });
    const previous = queryClient.getQueryData(["posts"]);
    // Optimistically add the post to the cache
    queryClient.setQueryData(["posts"], old => [...old, newPost]);
    return { previous }; // context for rollback
  },

  onError: (err, newPost, context) => {
    // Roll back on failure
    queryClient.setQueryData(["posts"], context.previous);
  },

  onSettled: () => {
    queryClient.invalidateQueries({ queryKey: ["posts"] }); // sync with server
  },
});
```

---

**9. How do you handle paginated API calls in React?**

Two approaches:

```jsx
// Approach 1: simple page state
const [page, setPage] = useState(1);
const { data } = useFetch(`/api/posts?page=${page}&limit=10`);

// Approach 2: React Query useInfiniteQuery
const { data, fetchNextPage, hasNextPage } = useInfiniteQuery({
  queryKey: ["posts"],
  queryFn: ({ pageParam = 1 }) =>
    fetch(`/api/posts?page=${pageParam}`).then(r => r.json()),
  getNextPageParam: (lastPage, pages) => lastPage.nextCursor,
});
```

---

**10. What happens if you forget the dependency array in `useEffect`?**

Without `[]`, the effect runs after every render — including renders caused by the `setState` inside it, creating an infinite loop of fetches.

```jsx
// WRONG — runs after every render, infinite fetch loop!
useEffect(() => {
  fetch("/api/data").then(r => r.json()).then(setData);
});

// CORRECT — runs once on mount
useEffect(() => {
  fetch("/api/data").then(r => r.json()).then(setData);
}, []);

// CORRECT — re-runs only when userId changes
useEffect(() => {
  fetch(`/api/users/${userId}`).then(r => r.json()).then(setUser);
}, [userId]);
```
