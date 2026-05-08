- Category: React Hooks
- Difficulty: Beginner
- Related: useState, useRef, hooks, component-lifecycle

### useEffect — Run Side Effects After Every Render
`useEffect` lets you perform side effects in functional components — things that happen outside the render itself, like fetching data, subscribing to events, starting timers, or manually changing the DOM. It runs after React has updated the screen.

**Analogy**
A morning alarm. After you wake up (after render), the alarm triggers your routine — brew coffee, check emails (side effects). You can set the alarm to ring every morning (no dependency array), only once when you move in (empty `[]`), or only when your schedule changes (`[schedule]`). The alarm has an "off" button for when you leave the house (cleanup).

---

### 1. Three Forms — Dependency Array Controls When It Runs

**Theory**
The second argument to `useEffect` — the dependency array — controls when the effect re-runs:
- **No array** → runs after every render
- **`[]` empty array** → runs once after the first render (mount)
- **`[dep1, dep2]`** → runs after the first render and again whenever any dep changes

**Working Flow**

![flow-chart](flow-chart.png)

**Example**
```jsx
import { useState, useEffect } from 'react';

function EffectDemo() {
  const [count, setCount] = useState(0);
  const [name, setName]   = useState('');

  // Runs after EVERY render
  useEffect(() => {
    console.log('After every render');
  });

  // Runs ONCE on mount
  useEffect(() => {
    console.log('Mounted!');
  }, []);

  // Runs when count changes
  useEffect(() => {
    console.log('count changed to:', count);
  }, [count]);

  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>
      <input value={name} onChange={e => setName(e.target.value)} />
    </div>
  );
}
```

**Output**
```
// On first render:
After every render
Mounted!
count changed to: 0

// Click +1 button:
After every render
count changed to: 1

// Type in input (name changes, count doesn't):
After every render
(count effect does NOT run — only count changed would trigger it)
```

---

### 2. Cleanup Function — Preventing Memory Leaks

**Theory**
The function returned from `useEffect` is the cleanup function. React runs it when:
1. The component unmounts (removed from DOM)
2. Before running the effect again (if deps changed)

This prevents memory leaks from event listeners, timers, and subscriptions left running after the component is gone.

**Working Flow**

![flow-chart-2](flow-chart-2.png)

**Example**
```jsx
import { useState, useEffect } from 'react';

function Timer() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    console.log('Starting timer');

    const id = setInterval(() => {
      setSeconds(s => s + 1);
    }, 1000);

    // Cleanup — runs on unmount OR before next effect
    return () => {
      console.log('Clearing timer');
      clearInterval(id);
    };
  }, []); // only starts once on mount

  return <p>Elapsed: {seconds}s</p>;
}
```

**Output**
```
// Mount:
Starting timer

// Every second:
Elapsed: 1s
Elapsed: 2s
...

// Unmount (component removed):
Clearing timer   ← cleanup prevents the interval from leaking
```

---

### 3. Fetching Data — Loading / Error / Data Pattern

**Theory**
The most common use of `useEffect` is fetching data after a component mounts. Always handle three states: loading, error, and data. Always include cleanup (AbortController) to cancel in-flight requests if the component unmounts before the fetch completes.

**Working Flow**

![flow-chart-3](flow-chart-3.png)

**Example**
```jsx
import { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser]       = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError]     = useState(null);

  useEffect(() => {
    const controller = new AbortController();

    setLoading(true);
    setError(null);

    fetch(`https://jsonplaceholder.typicode.com/users/${userId}`, {
      signal: controller.signal,
    })
      .then(res => {
        if (!res.ok) throw new Error('User not found');
        return res.json();
      })
      .then(data => {
        setUser(data);
        setLoading(false);
      })
      .catch(err => {
        if (err.name === 'AbortError') return; // ignore cancelled requests
        setError(err.message);
        setLoading(false);
      });

    // Cleanup — cancel fetch if userId changes or component unmounts
    return () => controller.abort();
  }, [userId]);

  if (loading) return <p>Loading...</p>;
  if (error)   return <p>Error: {error}</p>;
  return <p>{user?.name}</p>;
}
```

**Output**
```
// Mount with userId=1:
Loading...      ← while fetch is in progress
Leanne Graham   ← after fetch completes

// userId prop changes to 2:
Loading...      ← previous fetch cancelled, new one starts
Ervin Howell    ← new data shown
```

---

### 4. Event Listeners — Add on Mount, Remove on Unmount

**Theory**
Adding event listeners to `window` or `document` inside `useEffect` is a common pattern. The cleanup function removes the listener to prevent it from firing after the component is gone — or from being added multiple times on re-renders.

**Working Flow**

![flow-chart-4](flow-chart-4.png)

**Example**
```jsx
import { useState, useEffect } from 'react';

function KeyTracker() {
  const [lastKey, setLastKey] = useState('');

  useEffect(() => {
    function handleKeyDown(e) {
      setLastKey(e.key);
    }

    window.addEventListener('keydown', handleKeyDown);

    return () => {
      window.removeEventListener('keydown', handleKeyDown);
    };
  }, []); // add listener once on mount, remove on unmount

  return <p>Last key pressed: {lastKey || 'none'}</p>;
}
```

**Output**
```
// Mount:
Last key pressed: none

// Press 'a':
Last key pressed: a

// Press 'Enter':
Last key pressed: Enter

// Unmount:
← listener is removed, no memory leak
```

---

### 5. Common Mistakes

**Theory**
Three mistakes cause most `useEffect` bugs:
1. **Missing deps** → stale closure reads old values
2. **Infinite loop** → setting state inside effect with that state as dep
3. **Async directly** → `useEffect` cannot be `async`

**Working Flow**

![flow-chart-5](flow-chart-5.png)

**Example**
```jsx
// ❌ MISTAKE 1 — Missing dependency (stale closure)
useEffect(() => {
  console.log(count); // always logs the initial value, never updates
}, []); // count is used but not listed

// ✅ FIX — add count to deps
useEffect(() => {
  console.log(count);
}, [count]);

// ❌ MISTAKE 2 — Infinite loop
const [data, setData] = useState([]);
useEffect(() => {
  setData([...data, 'new']);  // sets data → triggers effect → sets data → ♾️
}, [data]);

// ✅ FIX — use functional update or remove dep
useEffect(() => {
  setData(prev => [...prev, 'new']);
}, []); // runs once

// ❌ MISTAKE 3 — async effect
useEffect(async () => {           // ← async useEffect returns a Promise, not a cleanup fn
  const data = await fetch('/api');
}, []);

// ✅ FIX — define async function inside effect
useEffect(() => {
  async function load() {
    const res  = await fetch('/api');
    const data = await res.json();
    setData(data);
  }
  load();
}, []);
```

---

### 6. Real-World — Live Search with Debounce

**Working Flow**

![flow-chart-6](flow-chart-6.png)

**Example**
```jsx
import { useState, useEffect } from 'react';

function LiveSearch() {
  const [query, setQuery]     = useState('');
  const [results, setResults] = useState([]);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    if (!query.trim()) {
      setResults([]);
      return;
    }

    setLoading(true);

    // Debounce — wait 400ms after user stops typing
    const timer = setTimeout(async () => {
      const controller = new AbortController();
      try {
        const res  = await fetch(
          `https://jsonplaceholder.typicode.com/users?name_like=${query}`,
          { signal: controller.signal }
        );
        const data = await res.json();
        setResults(data);
      } finally {
        setLoading(false);
      }
    }, 400);

    // Cleanup — cancel timer if query changes before 400ms
    return () => clearTimeout(timer);
  }, [query]);

  return (
    <div>
      <input
        value={query}
        onChange={e => setQuery(e.target.value)}
        placeholder="Search users..."
      />
      {loading && <p>Searching...</p>}
      <ul>
        {results.map(u => (
          <li key={u.id}>{u.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

---

[View Interview Questions](./interview.md)
