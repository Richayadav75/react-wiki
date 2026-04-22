- Category: Hooks
- Difficulty: Beginner
- Related: useLayoutEffect, useState, useRef

`useEffect` lets you **synchronize a component with an external system** — fetching data, subscribing to events, manipulating the DOM, or setting up timers. It runs *after* the browser has painted.

## Syntax

```tsx
useEffect(() => {
  // setup code (runs after render)
  return () => {
    // cleanup code (runs before next effect or unmount)
  };
}, [dependency1, dependency2]);
```

## Dependency array rules

| Array | Behaviour |
|-------|-----------|
| Omitted | Runs after **every** render |
| `[]` | Runs **once** after mount |
| `[a, b]` | Runs when `a` or `b` changes |

## Example — Fetch on mount

```tsx
import { useState, useEffect } from 'react';

function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    let cancelled = false;

    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => {
        if (!cancelled) setUser(data);
      });

    return () => { cancelled = true; }; // cleanup for fast refresh / prop change
  }, [userId]);

  if (!user) return <p>Loading…</p>;
  return <p>{user.name}</p>;
}
```

## Example — Event listener

```tsx
useEffect(() => {
  const handler = () => setScrollY(window.scrollY);
  window.addEventListener('scroll', handler);
  return () => window.removeEventListener('scroll', handler);
}, []);
```

## Common Pitfalls

- **Stale closures** — If your effect reads a value that changes, add it to the dependency array.
- **Infinite loops** — Putting an object/array literal or inline function in the deps array creates a new reference on every render.
- **Missing cleanup** — Subscriptions and timers that aren't cleaned up cause memory leaks.
- **Setting state unconditionally in an effect** — Always check if the component is still mounted (use the cancellation pattern above).
- **Using `async` directly** — `useEffect(async () => …)` is not supported. Declare an `async` function inside and call it.

```tsx
// ✅ Correct async pattern
useEffect(() => {
  async function load() {
    const data = await fetchData();
    setData(data);
  }
  load();
}, []);
```

## Learn More

- [React Docs — useEffect](https://react.dev/reference/react/useEffect)
- [A Complete Guide to useEffect — Dan Abramov](https://overreacted.io/a-complete-guide-to-useeffect/)
- [You Might Not Need an Effect](https://react.dev/learn/you-might-not-need-an-effect)
