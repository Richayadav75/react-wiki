- Category: Performance
- Difficulty: Intermediate
- Related: useCallback, React.memo

`useMemo` **caches the result of a computation** between renders. React only re-runs the function when one of the listed dependencies changes. Use it to avoid expensive recalculations on every render.

## Syntax

```tsx
const cachedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

## Example — Expensive filter

```tsx
import { useMemo, useState } from 'react';

function ProductList({ products }: { products: Product[] }) {
  const [query, setQuery] = useState('');

  // Only re-filtered when products or query changes
  const filtered = useMemo(
    () =>
      products.filter(p =>
        p.name.toLowerCase().includes(query.toLowerCase())
      ),
    [products, query]
  );

  return (
    <>
      <input value={query} onChange={e => setQuery(e.target.value)} />
      {filtered.map(p => <ProductCard key={p.id} product={p} />)}
    </>
  );
}
```

## Example — Stable object reference for context

```tsx
const contextValue = useMemo(
  () => ({ user, logout }),
  [user, logout]
);

return <AuthContext.Provider value={contextValue}>{children}</AuthContext.Provider>;
```

## When NOT to use useMemo

`useMemo` has overhead itself. Skip it when:
- The computation is cheap (arithmetic, simple array operations)
- The dependencies change on almost every render anyway
- You're optimising prematurely — profile first

## useMemo vs useCallback

| Hook | Caches |
|------|--------|
| `useMemo` | The **return value** of a function |
| `useCallback` | The **function itself** |

`useCallback(fn, deps)` is equivalent to `useMemo(() => fn, deps)`.

## Common Pitfalls

- **Stale dependencies** — If you forget a dependency, you'll read outdated values. Use the eslint-plugin-react-hooks exhaustive-deps rule.
- **Memoizing everything** — Adds cognitive overhead; reserve for genuinely expensive calculations.
- **Reference equality trap** — Object/array literals in deps (`[{ id: 1 }]`) are always new references; lift them out or use a stable primitive.

## Learn More

- [React Docs — useMemo](https://react.dev/reference/react/useMemo)
- [Before you memo() — Dan Abramov](https://overreacted.io/before-you-memo/)
