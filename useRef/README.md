- Category: Hooks
- Difficulty: Intermediate
- Related: useState, forwardRef

`useRef` returns a **mutable ref object** whose `.current` property persists across renders without triggering a re-render when changed. Its two main uses are: **accessing DOM nodes** and **storing mutable values** that shouldn't cause re-renders.

## Syntax

```tsx
const ref = useRef<Type>(initialValue);
// Access via ref.current
```

## Use case 1 — DOM access

```tsx
import { useRef, useEffect } from 'react';

function AutoFocusInput() {
  const inputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    inputRef.current?.focus();
  }, []);

  return <input ref={inputRef} />;
}
```

## Use case 2 — Storing previous values

```tsx
function usePrevious<T>(value: T) {
  const ref = useRef<T | undefined>(undefined);

  useEffect(() => {
    ref.current = value;
  }, [value]);

  return ref.current; // the value from the previous render
}
```

## Use case 3 — Stable callback reference (interval example)

```tsx
function Timer() {
  const [count, setCount] = useState(0);
  const callbackRef = useRef(() => {});

  // Keep ref in sync with latest callback without restarting the interval
  callbackRef.current = () => setCount(c => c + 1);

  useEffect(() => {
    const id = setInterval(() => callbackRef.current(), 1000);
    return () => clearInterval(id);
  }, []); // empty deps — interval never restarts

  return <p>{count}</p>;
}
```

## ref vs state

| | `useRef` | `useState` |
|---|---|---|
| Triggers re-render | ❌ No | ✅ Yes |
| Persists across renders | ✅ Yes | ✅ Yes |
| Can hold any value | ✅ Yes | ✅ Yes |
| Synchronous read | ✅ Yes | ❌ No (async update) |

## Common Pitfalls

- **Reading `.current` during render** — The value is mutable and not tracked by React; rendering `ref.current` directly can show stale values.
- **Not using `null` as initial value for DOM refs** — TypeScript will complain if you don't use `useRef<HTMLElement>(null)`.
- **Mutating `.current` and expecting a re-render** — If you need the UI to update, use `useState` instead.

## Learn More

- [React Docs — useRef](https://react.dev/reference/react/useRef)
- [Referencing Values with Refs](https://react.dev/learn/referencing-values-with-refs)
