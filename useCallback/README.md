- Category: Performance
- Difficulty: Intermediate
- Related: useMemo, React.memo

`useCallback` returns a **memoized function** — the same function reference between renders unless a dependency changes. It's primarily useful when passing callbacks to child components wrapped in `React.memo`, preventing unnecessary re-renders.

## Syntax

```tsx
const stableCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]);
```

## Example — Stable handler for a memoized child

```tsx
import { useCallback, memo, useState } from 'react';

const Button = memo(({ onClick, label }: { onClick: () => void; label: string }) => {
  console.log('Button rendered');
  return <button onClick={onClick}>{label}</button>;
});

function Parent() {
  const [count, setCount] = useState(0);
  const [text, setText] = useState('');

  // Without useCallback: new reference on every render → Button always re-renders
  // With useCallback: same reference unless deps change → Button is stable
  const handleClick = useCallback(() => {
    setCount(c => c + 1);
  }, []); // no deps: count updated via functional update

  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <Button onClick={handleClick} label={`Clicked ${count} times`} />
    </>
  );
}
```

## Example — useCallback in a custom hook

```tsx
function useSearch(items: string[]) {
  const [query, setQuery] = useState('');

  const search = useCallback(
    (q: string) => items.filter(i => i.includes(q)),
    [items]
  );

  return { query, setQuery, search };
}
```

## When NOT to use useCallback

- The child is **not** wrapped in `React.memo` (no benefit)
- The function is re-created with new dependencies every render anyway
- The function is only used inside the same component (not passed as prop)

## Common Pitfalls

- **Forgetting to add deps** — Using a stale closed-over variable that never updates.
- **Overusing it** — Memoizing trivial handlers adds noise without measurable gains. Profile first.
- **Missing React.memo on the child** — `useCallback` alone won't stop a child from re-rendering.

## Learn More

- [React Docs — useCallback](https://react.dev/reference/react/useCallback)
- [When to useMemo and useCallback — Kent C. Dodds](https://kentcdodds.com/blog/usememo-and-usecallback)

---

[View Interview Questions](./interview.md)
