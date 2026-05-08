- Category: React Performance
- Difficulty: Intermediate
- Related: useCallback, react-memo, hooks

### useMemo — Cache Expensive Computed Values Between Renders
`useMemo` is a React hook that memoizes the result of a calculation. It runs the calculation on the first render, caches the result, and only re-runs it when a listed dependency changes. On every other render it returns the cached value instantly.

**Analogy**
A chef who pre-makes a dish and stores it in the fridge. Every time a customer orders it, instead of cooking from scratch, they plate the stored dish — unless a key ingredient changes. Only then do they cook fresh.

---

### 1. The Problem — Recalculating on Every Render

**Theory**
React re-runs the entire component function on every render. Any calculation inside the component runs every render too — even if the inputs haven't changed. For cheap operations (adding two numbers) this is fine. For expensive ones (filtering 10,000 items, sorting large arrays), re-running on every render wastes CPU and causes lag.

**Working Flow**

![flow-chart](flow-chart.png)

**Example**
```jsx
import { useState } from 'react';

function NumberList() {
  const [count, setCount]   = useState(5);
  const [theme, setTheme]   = useState('light');

  // This runs on EVERY render — even just toggling the theme
  const numbers = Array.from({ length: count }, (_, i) => i + 1);
  const sum     = numbers.reduce((acc, n) => acc + n, 0);

  return (
    <div className={theme}>
      <button onClick={() => setCount(c => c + 1)}>Add Number</button>
      <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>
        Toggle Theme
      </button>
      <p>Numbers: {numbers.join(', ')}</p>
      <p>Sum: {sum}</p>
    </div>
  );
}
```

**Output**
```
// Toggling theme also recomputes numbers and sum — wasteful!
// Imagine count = 100,000 — every theme toggle would freeze the UI.
```

---

### 2. With useMemo — Calculation Skipped When Deps Unchanged

**Theory**
Wrap the expensive calculation in `useMemo(() => result, [deps])`. React caches the return value and only recalculates when a dependency changes. Unrelated state changes (like theme) no longer trigger the calculation.

**Working Flow**

![flow-chart-2](flow-chart-2.png)

**Example**
```jsx
import { useState, useMemo } from 'react';

function filterProducts(products, query) {
  console.log('Filtering...');  // watch how often this fires
  return products.filter(p =>
    p.name.toLowerCase().includes(query.toLowerCase())
  );
}

const ALL_PRODUCTS = Array.from({ length: 1000 }, (_, i) => ({
  id: i,
  name: `Product ${i}`,
  price: Math.floor(Math.random() * 500),
}));

function ProductSearch() {
  const [query, setQuery]   = useState('');
  const [darkMode, setDark] = useState(false);

  // ✅ Only re-filters when query changes — darkMode is NOT a dep
  const filtered = useMemo(
    () => filterProducts(ALL_PRODUCTS, query),
    [query]
  );

  return (
    <div style={{ background: darkMode ? '#333' : '#fff' }}>
      <input
        value={query}
        onChange={e => setQuery(e.target.value)}
        placeholder="Search products..."
      />
      <button onClick={() => setDark(d => !d)}>Toggle Dark Mode</button>
      <p>{filtered.length} results</p>
    </div>
  );
}
```

**Output**
```
// Toggling dark mode:
(silence)      ← cached result returned, no Filtering... log

// Changing query:
Filtering...   ← correctly re-runs because query changed
```

---

### 3. Referential Stability — Memoizing Objects and Arrays

**Theory**
Objects and arrays created inside a component are new references on every render. When passed as props to a `React.memo` child, the child always re-renders because `{} !== {}`. `useMemo` returns the same reference across renders, so `React.memo` can do its job.

**Working Flow**

![flow-chart-3](flow-chart-3.png)

**Example**
```jsx
import { useState, useMemo, memo } from 'react';

const Chart = memo(function Chart({ data }) {
  console.log('Chart rendered');
  return <div>Chart with {data.length} points</div>;
});

function Dashboard() {
  const [theme, setTheme] = useState('light');
  const [count, setCount] = useState(10);

  // ❌ Without useMemo — new array every render → Chart always re-renders
  // const chartData = Array.from({ length: count }, (_, i) => i * 2);

  // ✅ With useMemo — same array reference unless count changes
  const chartData = useMemo(
    () => Array.from({ length: count }, (_, i) => i * 2),
    [count]
  );

  return (
    <div>
      <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>
        Toggle Theme
      </button>
      <button onClick={() => setCount(c => c + 1)}>
        Add Data Point
      </button>
      <Chart data={chartData} />
    </div>
  );
}
```

**Output**
```
// Initial:
Chart rendered

// Toggle Theme (with useMemo):
(silence)       ← chartData reference unchanged → React.memo blocks re-render

// Add Data Point:
Chart rendered  ← count changed → useMemo recalculates → new reference
```

---

### 4. Dependency Array — When It Recalculates

**Theory**
`useMemo` compares each dependency using `Object.is`. The value recalculates only when at least one dependency changes.
- `[]` → computes once, never again
- `[dep]` → recomputes whenever dep changes
- No array → computes every render (defeats the purpose)

**Working Flow**

![flow-chart-4](flow-chart-4.png)

**Example**
```jsx
import { useState, useMemo } from 'react';

function Stats({ scores }) {
  const [label, setLabel] = useState('Statistics');

  const stats = useMemo(() => {
    console.log('Computing stats...');
    const total = scores.reduce((s, n) => s + n, 0);
    const avg   = (total / scores.length).toFixed(2);
    const max   = Math.max(...scores);
    const min   = Math.min(...scores);
    return { total, avg, max, min };
  }, [scores]);   // recalculates only when scores array changes

  return (
    <div>
      <input
        value={label}
        onChange={e => setLabel(e.target.value)}
      />
      <p>
        {label}: Total={stats.total} | Avg={stats.avg} | Max={stats.max}
      </p>
    </div>
  );
}
```

**Output**
```
// Typing in label input (label state changes, scores doesn't):
(silence)          ← stats are cached, Computing stats... not logged

// When scores prop changes:
Computing stats... ← dependency changed, recalculates fresh
```

---

### 5. useMemo vs useCallback

**Theory**
Both are memoization hooks — the difference is what they cache.

| Hook | Caches | Returns |
| :--- | :--- | :--- |
| `useMemo` | Result of calling a function | The **value** the function returned |
| `useCallback` | The function itself | The **function** (not its result) |

**Working Flow**

![flow-chart-5](flow-chart-5.png)

**Example**
```jsx
import { useMemo, useCallback } from 'react';

function Example({ items, query }) {
  // useMemo — cache a VALUE (the filtered array)
  const filteredItems = useMemo(
    () => items.filter(item => item.includes(query)),
    [items, query]
  );

  // useCallback — cache a FUNCTION (the handler)
  const handleSelect = useCallback(
    (item) => {
      console.log('Selected:', item);
    },
    []
  );

  return (
    <ul>
      {filteredItems.map(item => (
        <li key={item} onClick={() => handleSelect(item)}>
          {item}
        </li>
      ))}
    </ul>
  );
}
```

---

### 6. When NOT to Use useMemo

**Theory**
`useMemo` has overhead: React stores the cached value, runs dependency comparison every render, and manages the dependency list. For cheap operations this overhead may cost more than just re-running the calculation.

**Working Flow**

![flow-chart-6](flow-chart-6.png)

**Example**
```jsx
// ❌ Over-memoizing — adds cost without benefit
const doubled  = useMemo(() => count * 2, [count]);
const greeting = useMemo(() => `Hello, ${name}!`, [name]);

// ✅ Just write it directly
const doubled  = count * 2;
const greeting = `Hello, ${name}!`;

// ✅ Worth memoizing — genuinely expensive
const sortedUsers = useMemo(
  () => [...users].sort((a, b) => a.name.localeCompare(b.name)),
  [users]
);
```

**Rule:** Only use `useMemo` when:
- Calculation is demonstrably slow (confirm with React DevTools Profiler)
- You need a stable reference to prevent child re-renders

---

[View Interview Questions](./interview.md)
