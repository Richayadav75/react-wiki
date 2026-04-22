- Category: Patterns
- Difficulty: Intermediate
- Related: useState, useEffect, useRef

A **custom hook** is a JavaScript function whose name starts with `use` and that calls other React hooks. It lets you **extract and reuse stateful logic** between components without changing the component tree (no HOCs, no render props).

## Rules

1. Name starts with `use` (e.g. `useLocalStorage`, `useDebounce`)
2. Can call any other hook inside
3. Stateful logic is isolated — each component calling the hook gets its own state

## Example — useDebounce

```tsx
import { useState, useEffect } from 'react';

function useDebounce<T>(value: T, delay = 300): T {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}

// Usage
function SearchBar() {
  const [query, setQuery] = useState('');
  const debouncedQuery = useDebounce(query, 500);

  useEffect(() => {
    if (debouncedQuery) fetchResults(debouncedQuery);
  }, [debouncedQuery]);

  return <input value={query} onChange={e => setQuery(e.target.value)} />;
}
```

## Example — useLocalStorage

```tsx
import { useState } from 'react';

function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });

  const setValue = (value: T | ((prev: T) => T)) => {
    const valueToStore = value instanceof Function ? value(storedValue) : value;
    setStoredValue(valueToStore);
    window.localStorage.setItem(key, JSON.stringify(valueToStore));
  };

  return [storedValue, setValue] as const;
}
```

## Example — useFetch

```tsx
import { useState, useEffect } from 'react';

function useFetch<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    let cancelled = false;
    setLoading(true);

    fetch(url)
      .then(res => res.json())
      .then(d => { if (!cancelled) { setData(d); setLoading(false); } })
      .catch(e => { if (!cancelled) { setError(e.message); setLoading(false); } });

    return () => { cancelled = true; };
  }, [url]);

  return { data, loading, error };
}
```

## Common Pitfalls

- **Not starting the name with `use`** — React's linter won't enforce hook rules for it.
- **Calling hooks conditionally inside the custom hook** — The same rules of hooks apply inside.
- **Returning too much** — Keep the API surface minimal and focused on one concern.

## Learn More

- [React Docs — Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)
- [useHooks.com](https://usehooks.com/) — collection of production-ready custom hooks
