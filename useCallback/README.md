- Category: React Performance
- Difficulty: Intermediate
- Related: useMemo, react-memo, hooks

`useCallback` is a React hook that returns a **memoized (cached) version of a function**. React normally creates a brand-new function object on every render. useCallback skips that re-creation and hands back the same function reference — as long as the dependencies haven't changed.

**Analogy**
Imagine you print a flyer for your shop. Every time a customer visits, instead of printing a fresh flyer from scratch, you hand them the same printed copy from your drawer — unless the offer changes. useCallback is that drawer. It keeps the same function "flyer" until something meaningful changes.

---

### 1. What useCallback Does — Memoizing a Function Reference

**Theory**
In JavaScript, every function literal creates a new object in memory. So `() => doSomething()` written inside a component produces a *new* reference on every render — even if the code is identical. This matters because child components compare props by reference. A new function reference = a changed prop = a re-render of the child, even if the logic is the same.

`useCallback(fn, [deps])` tells React: "Cache this function. Give it back unchanged until one of the deps changes."

**Working Flow**

![flow-chart](flow-chart.png)

**Example**
```jsx
import { useState, useCallback } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  const [text, setText] = useState('');

  // Without useCallback — new function every render
  const incrementRaw = () => setCount(c => c + 1);

  // With useCallback — same reference every render (no deps)
  const incrementStable = useCallback(() => {
    setCount(c => c + 1);
  }, []);

  return (
    <div>
      <input value={text} onChange={e => setText(e.target.value)} />
      <p>Count: {count}</p>
      <button onClick={incrementStable}>Increment</button>
    </div>
  );
}
```

**Output**
```
// Typing in the input triggers a re-render of Counter.
// incrementRaw  → new function reference every re-render
// incrementStable → SAME reference every re-render (deps = [])
// Result: stable reference, avoids unnecessary child updates
```

**Explanation**
`incrementStable` is created once. Even when `text` state changes and the component re-renders, `incrementStable` points to the exact same object in memory. `incrementRaw` would be a new object every single time.

---

### 2. Without vs With useCallback — Seeing Unnecessary Re-renders

**Theory**
The problem becomes visible when you pass a callback as a prop to a child component. Without useCallback, every parent re-render gives the child a new function prop. React sees "prop changed" and re-renders the child — even if nothing functional changed.

**Working Flow**

![flow-chart](flow-chart-2.png)

**Example**
```jsx
import { useState, useCallback, memo } from 'react';

// Child wrapped in React.memo — only re-renders if props change
const SaveButton = memo(({ onSave }) => {
  console.log('SaveButton rendered'); // watch the console!
  return <button onClick={onSave}>Save</button>;
});

// WITHOUT useCallback
function FormBad() {
  const [name, setName] = useState('');

  // New function every render → SaveButton always re-renders
  const handleSave = () => {
    console.log('Saving:', name);
  };

  return (
    <>
      <input value={name} onChange={e => setName(e.target.value)} />
      <SaveButton onSave={handleSave} />
    </>
  );
}

// WITH useCallback
function FormGood() {
  const [name, setName] = useState('');

  // Same reference → SaveButton re-renders ONLY when name changes
  const handleSave = useCallback(() => {
    console.log('Saving:', name);
  }, [name]);

  return (
    <>
      <input value={name} onChange={e => setName(e.target.value)} />
      <SaveButton onSave={handleSave} />
    </>
  );
}
```

**Output**
```
// FormBad: type one character in input
SaveButton rendered   ← fires on every keystroke (wasteful)

// FormGood: type one character in input
SaveButton rendered   ← fires only when name actually changes
                        (same behaviour here, but if other state
                         changed in FormGood, SaveButton stays quiet)
```

**Explanation**
`React.memo` does a shallow comparison of props. For functions, shallow comparison is a reference check. Without `useCallback`, the reference is always new → memo is bypassed. With `useCallback`, memo works as intended.

---

### 3. useCallback + React.memo — They Work Together

**Theory**
`useCallback` and `React.memo` are a pair. Neither is fully effective without the other:
- `useCallback` alone: stabilizes the function reference in the parent — no direct rendering benefit if the child doesn't use memo.
- `React.memo` alone: the child bails on re-renders, but if the function prop is always a new reference, memo's check always fails.
- **Together**: parent holds a stable function reference + child skips re-renders when that reference hasn't changed. Perfect optimization.

**Working Flow**

![flow-chart](flow-chart-3.png)

**Example**
```jsx
import { useState, useCallback, memo } from 'react';

const ProductRow = memo(({ product, onDelete }) => {
  console.log('ProductRow rendered:', product.name);
  return (
    <div>
      <span>{product.name} — ₹{product.price}</span>
      <button onClick={() => onDelete(product.id)}>Delete</button>
    </div>
  );
});

function ProductList() {
  const [products, setProducts] = useState([
    { id: 1, name: 'Phone', price: 800 },
    { id: 2, name: 'Watch', price: 200 },
  ]);
  const [filter, setFilter] = useState('');

  // Stable reference — ProductRow won't re-render just because filter changes
  const handleDelete = useCallback((id) => {
    setProducts(prev => prev.filter(p => p.id !== id));
  }, []); // no deps needed — uses functional updater

  return (
    <>
      <input
        placeholder="Filter..."
        value={filter}
        onChange={e => setFilter(e.target.value)}
      />
      {products.map(p => (
        <ProductRow key={p.id} product={p} onDelete={handleDelete} />
      ))}
    </>
  );
}
```

**Output**
```
// Initial render:
ProductRow rendered: Phone
ProductRow rendered: Watch

// Typing in filter input (only filter state changes):
// No "ProductRow rendered" logs — memo + useCallback blocked re-renders!

// Clicking Delete on Phone:
ProductRow rendered: Watch   ← only Watch re-renders (products state changed)
```

**Explanation**
Typing in the filter input changes `filter` state, re-rendering `ProductList`. But `handleDelete` is the same reference (no deps changed), so `ProductRow` components receive the same prop → `React.memo` blocks their re-render. The child only re-renders when `products` actually changes.

---

### 4. Dependency Array Rules — Stale Closures Explained

**Theory**
The dependency array `[dep1, dep2]` tells React when to create a fresh version of the function. If you list a variable that the function uses, React will rebuild the function whenever that variable changes, ensuring the function always "sees" the latest value.

**Stale closure**: If you use a variable inside `useCallback` but forget to list it in deps, the function will forever read the *old* value it captured on creation. This is one of the most common bugs with useCallback.

**Working Flow**

![flow-chart](flow-chart-4.png)

**Example**
```jsx
import { useState, useCallback } from 'react';

function Chat() {
  const [message, setMessage] = useState('');
  const [userId, setUserId] = useState('user-1');

  // ❌ STALE CLOSURE — userId is missing from deps
  const sendBad = useCallback(() => {
    console.log(`Sending "${message}" to ${userId}`);
    // userId will always be 'user-1' no matter what!
  }, [message]); // forgot userId

  // ✅ CORRECT — all used variables listed
  const sendGood = useCallback(() => {
    console.log(`Sending "${message}" to ${userId}`);
  }, [message, userId]);

  // ✅ BEST — use functional updater to avoid deps on state
  const appendMessage = useCallback((extra) => {
    setMessage(prev => prev + extra); // reads latest via updater, not closure
  }, []); // empty deps — stable forever

  return (
    <div>
      <input value={message} onChange={e => setMessage(e.target.value)} />
      <button onClick={() => setUserId('user-2')}>Switch User</button>
      <button onClick={sendGood}>Send</button>
    </div>
  );
}
```

**Output**
```
// sendBad after switching to user-2:
Sending "hello" to user-1   ← stale! userId never updated in closure

// sendGood after switching to user-2:
Sending "hello" to user-2   ← correct, fresh closure
```

**Explanation**
Always list every variable from the outer scope that the function reads or writes. ESLint's `exhaustive-deps` rule will catch these automatically if you have `eslint-plugin-react-hooks` configured.

---

### 5. useCallback Inside Custom Hooks

**Theory**
Custom hooks often need to return stable function references. Without useCallback inside the custom hook, every time the hook's consumer re-renders, they get a new function back — breaking any downstream memo optimizations.

**Working Flow**

![flow-chart](flow-chart-5.png)

**Example**
```jsx
import { useState, useCallback, useEffect } from 'react';

// Custom hook — useFetch with stable refetch function
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const fetchData = useCallback(async () => {
    setLoading(true);
    setError(null);
    try {
      const res = await fetch(url);
      const json = await res.json();
      setData(json);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }, [url]); // re-creates only when url changes

  useEffect(() => {
    fetchData();
  }, [fetchData]); // safe because fetchData is stable

  return { data, loading, error, refetch: fetchData };
}

// Custom hook — useDebounce with stable handler
function useDebounce(callback, delay) {
  const callbackRef = useCallback(callback, [callback]);

  const debouncedFn = useCallback((...args) => {
    const timer = setTimeout(() => callbackRef(...args), delay);
    return () => clearTimeout(timer);
  }, [callbackRef, delay]);

  return debouncedFn;
}

// Usage
function SearchPage() {
  const [query, setQuery] = useState('');
  const { data, loading, refetch } = useFetch(`/api/search?q=${query}`);

  return (
    <div>
      <input value={query} onChange={e => setQuery(e.target.value)} />
      <button onClick={refetch}>Reload</button>
      {loading ? <p>Loading...</p> : <pre>{JSON.stringify(data)}</pre>}
    </div>
  );
}
```

**Output**
```
// When query changes → fetchData recreated (url changed) → useEffect fires → new fetch
// When other state changes → fetchData is SAME reference → useEffect does NOT re-fire
// refetch button → calls same stable fetchData function
```

**Explanation**
Returning `fetchData` as `refetch` from the custom hook gives consumers a stable function they can safely put in their own dependency arrays or pass to memoized children.

---

### 6. When NOT to Use useCallback — Anti-patterns

**Theory**
useCallback has a cost: React must store the function, track the dependency array, and run comparison logic on every render. If there's no memoized child consuming the function, or if the function changes on every render anyway, useCallback adds overhead with zero benefit.

**Working Flow**

![flow-chart](flow-chart-6.png)

**Example**
```jsx
import { useState, useCallback } from 'react';

function BadUsage() {
  const [count, setCount] = useState(0);

  // ❌ Anti-pattern 1: No memoized child receiving this — pointless
  const handleClick = useCallback(() => {
    setCount(c => c + 1);
  }, []);

  // ❌ Anti-pattern 2: Deps change every render — never actually cached
  const items = [1, 2, 3]; // new array every render
  const processItems = useCallback(() => {
    return items.map(i => i * 2);
  }, [items]); // items is always new → function always recreated

  // ❌ Anti-pattern 3: Wrapping an already-stable external function
  const log = useCallback(console.log, []); // console.log never changes!

  return <button onClick={handleClick}>Count: {count}</button>;
}

// ✅ Correct usage — only when child is memoized
const Display = memo(({ onClick }) => (
  <button onClick={onClick}>Click me</button>
));

function GoodUsage() {
  const [count, setCount] = useState(0);

  const handleClick = useCallback(() => {
    setCount(c => c + 1);
  }, []); // justified — memoized child receives this

  return <Display onClick={handleClick} />;
}
```

**Output**
```
BadUsage:
- handleClick: memoized but no benefit (no memo child)
- processItems: deps change every render → recreated anyway
- log: wraps a stable function unnecessarily

GoodUsage:
- handleClick: justified, Display is memo-wrapped
- Display: skips re-render when handleClick reference is stable
```

**Explanation**
The rule of thumb: only useCallback a function when (1) it is passed as a prop to a `React.memo`-wrapped child, or (2) it appears in the dependency array of a `useEffect` / `useMemo` / another `useCallback`. Everywhere else, it's noise.

---

### Real-World: Parent Passing Sort/Filter Callbacks to a Memoized List

```jsx
import { useState, useCallback, useMemo, memo } from 'react';

const ProductCard = memo(({ product }) => {
  console.log('ProductCard rendered:', product.name);
  return (
    <div className="card">
      <h3>{product.name}</h3>
      <p>₹{product.price} | {product.category}</p>
    </div>
  );
});

const ProductGrid = memo(({ products, onSort, onFilter }) => {
  return (
    <div>
      <button onClick={() => onSort('price')}>Sort by Price</button>
      <button onClick={() => onSort('name')}>Sort by Name</button>
      <input placeholder="Filter category" onChange={e => onFilter(e.target.value)} />
      <div className="grid">
        {products.map(p => <ProductCard key={p.id} product={p} />)}
      </div>
    </div>
  );
});

function ShopPage() {
  const [allProducts] = useState([
    { id: 1, name: 'Phone',  price: 800, category: 'Electronics' },
    { id: 2, name: 'Bag',    price:  50, category: 'Fashion'      },
    { id: 3, name: 'Watch',  price: 200, category: 'Electronics'  },
    { id: 4, name: 'Shoes',  price: 120, category: 'Fashion'      },
  ]);
  const [sortKey, setSortKey]       = useState('name');
  const [categoryFilter, setFilter] = useState('');

  // Stable callbacks — ProductGrid won't re-render due to these changing
  const handleSort   = useCallback((key) => setSortKey(key), []);
  const handleFilter = useCallback((val) => setFilter(val), []);

  const displayProducts = useMemo(() => {
    return [...allProducts]
      .filter(p => p.category.toLowerCase().includes(categoryFilter.toLowerCase()))
      .sort((a, b) => a[sortKey] > b[sortKey] ? 1 : -1);
  }, [allProducts, sortKey, categoryFilter]);

  return <ProductGrid products={displayProducts} onSort={handleSort} onFilter={handleFilter} />;
}
```

**Output**
```
// Initial render — all 4 cards log
ProductCard rendered: Bag
ProductCard rendered: Phone
ProductCard rendered: Shoes
ProductCard rendered: Watch

// Clicking "Sort by Price":
// sortKey changes → displayProducts recomputed → ProductGrid re-renders
// BUT handleSort and handleFilter are same reference → no extra renders from them

// Clicking "Sort by Name" again:
// Same result — grid re-renders only because products order changed
```

---

[View Interview Questions](./interview.md)
