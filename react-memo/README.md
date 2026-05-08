- Category: React Performance
- Difficulty: Intermediate
- Related: useCallback, useMemo, hooks

### React.memo — Skip Re-renders When Props Haven't Changed
`React.memo` is a higher-order component that wraps a functional component and tells React: "Only re-render this component if its props actually changed." If the parent re-renders but passes the same props, React reuses the last rendered output from memory.

**Analogy**
A photocopier with smart memory. If you copy the same document twice in a row, the second time it says "I already made this — here's the copy from the tray." It only goes back to the original when the document actually changes.

---

### 1. The Problem — Unnecessary Re-renders

**Theory**
When a parent component re-renders, every child component re-renders too — even if the child's props didn't change. For simple components this is harmless. For heavy components rendering large data, this wastes CPU on every keystroke, state toggle, or unrelated update.

**Working Flow**

![flow-chart](flow-chart.png)

**Example**
```jsx
import { useState } from 'react';

// Child — no memo
function Greeting({ name }) {
  console.log('Greeting rendered');
  return <h2>Hello, {name}!</h2>;
}

function App() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>
        Count: {count}
      </button>
      <Greeting name="Richa" />
    </div>
  );
}
```

**Output**
```
// Every button click:
Greeting rendered   ← fires even though name="Richa" never changed
Greeting rendered
Greeting rendered
```

**Explanation**
`Greeting` always gets `name="Richa"` — nothing changes for it. But because `App` re-renders on each click, React blindly re-runs `Greeting` too. `React.memo` fixes this.

---

### 2. With React.memo — Renders Blocked

**Theory**
Wrap the component with `React.memo()`. Before re-rendering, React does a shallow comparison of old vs new props. If all props are equal, it skips the re-render and reuses the previous output.

**Working Flow**

![flow-chart-2](flow-chart-2.png)

**Example**
```jsx
import { useState, memo } from 'react';

// Wrapped with memo — React compares props before re-rendering
const Greeting = memo(function Greeting({ name }) {
  console.log('Greeting rendered');
  return <h2>Hello, {name}!</h2>;
});

function App() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>
        Count: {count}
      </button>
      <Greeting name="Richa" />
    </div>
  );
}
```

**Output**
```
// Initial render:
Greeting rendered

// Every button click after:
(silence)   ← Greeting is NOT re-rendered
```

**Explanation**
React compares `"Richa" === "Richa"` → true → skip. The button click changes `count` (App re-renders) but Greeting's props are identical, so React reuses the last output. The component function is never called again.

---

### 3. The Shallow Comparison Trap — Objects and Functions

**Theory**
Shallow comparison checks if each prop value is the same reference. Objects and functions created inline are always new references even if the content looks identical. This breaks `React.memo` silently.

**Working Flow**

![flow-chart-3](flow-chart-3.png)

**Example**
```jsx
import { useState, memo } from 'react';

const Card = memo(function Card({ style, onClick }) {
  console.log('Card rendered');
  return (
    <div style={style}>
      <button onClick={onClick}>Click</button>
    </div>
  );
});

function App() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>Re-render App</button>

      {/* ❌ New object + new function every render → memo bypassed */}
      <Card
        style={{ color: 'red' }}
        onClick={() => console.log('clicked')}
      />
    </div>
  );
}
```

**Output**
```
// Every "Re-render App" click:
Card rendered   ← memo is bypassed! {} !== {} (different reference)
Card rendered
Card rendered
```

**Fix:** Stabilize with `useMemo` (object) and `useCallback` (function):
```jsx
const style    = useMemo(() => ({ color: 'red' }), []);
const onClick  = useCallback(() => console.log('clicked'), []);
// Now Card renders only once
```

---

### 4. Custom Comparison Function

**Theory**
When you need fine-grained control, pass a second argument to `React.memo` — a custom `areEqual(prevProps, nextProps)` function. Return `true` to skip re-render, `false` to allow it.

**Working Flow**

![flow-chart-4](flow-chart-4.png)

**Example**
```jsx
import { memo } from 'react';

const UserCard = memo(
  function UserCard({ user }) {
    console.log('UserCard rendered');
    return (
      <div>
        <p>{user.name}</p>
        <p>{user.role}</p>
      </div>
    );
  },
  // Only re-render if id or name changes — ignore role and avatar changes
  (prevProps, nextProps) => {
    return (
      prevProps.user.id   === nextProps.user.id &&
      prevProps.user.name === nextProps.user.name
    );
  }
);
```

**Output**
```
// Initial render with { id:1, name:"Alice", role:"Admin" }:
UserCard rendered

// Update to { id:1, name:"Alice", role:"Editor" } (role changed, id+name same):
(no re-render — areEqual returned true)

// Update to { id:1, name:"Bob", role:"Admin" } (name changed):
UserCard rendered   ← areEqual returned false
```

---

### 5. What React.memo Does NOT Prevent

**Theory**
React.memo only compares props. It cannot stop re-renders caused by:
- The component's own `useState` or `useReducer`
- A `useContext` value changing
- A parent's context changing

**Working Flow**

![flow-chart-5](flow-chart-5.png)

**Example**
```jsx
import { useState, useContext, createContext, memo } from 'react';

const ThemeContext = createContext('light');

const ThemedBox = memo(function ThemedBox() {
  const theme = useContext(ThemeContext);
  console.log('ThemedBox rendered');
  return <div className={theme}>I am {theme}</div>;
});

function App() {
  const [theme, setTheme] = useState('light');
  const [count, setCount] = useState(0);

  return (
    <ThemeContext.Provider value={theme}>
      <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>
        Toggle Theme
      </button>
      <button onClick={() => setCount(c => c + 1)}>
        Count: {count}
      </button>
      <ThemedBox />
    </ThemeContext.Provider>
  );
}
```

**Output**
```
// Toggle Theme:
ThemedBox rendered   ← context changed → memo cannot block this

// Count button:
(silence)            ← no prop/context change → memo blocks this
```

---

### 6. React.memo + useCallback + useMemo — Full Pattern

**Theory**
All three memoization tools work together. `React.memo` guards the child. `useCallback` stabilizes function props. `useMemo` stabilizes object/array props.

**Working Flow**

![flow-chart-6](flow-chart-6.png)

**Example**
```jsx
import { useState, useCallback, useMemo, memo } from 'react';

const ProductCard = memo(function ProductCard({ product, onDelete }) {
  console.log('ProductCard rendered:', product.name);
  return (
    <div>
      <h3>{product.name}</h3>
      <p>₹{product.price}</p>
      <button onClick={() => onDelete(product.id)}>Remove</button>
    </div>
  );
});

function Shop() {
  const [cart, setCart]     = useState([]);
  const [search, setSearch] = useState('');

  const products = [
    { id: 1, name: 'Phone', price: 800 },
    { id: 2, name: 'Watch', price: 200 },
  ];

  // useMemo — stable filtered array
  const filtered = useMemo(
    () => products.filter(p =>
      p.name.toLowerCase().includes(search.toLowerCase())
    ),
    [search]
  );

  // useCallback — stable function reference
  const handleDelete = useCallback(
    (id) => setCart(prev => prev.filter(i => i !== id)),
    []
  );

  return (
    <div>
      <input
        value={search}
        onChange={e => setSearch(e.target.value)}
        placeholder="Search..."
      />
      {filtered.map(p => (
        <ProductCard key={p.id} product={p} onDelete={handleDelete} />
      ))}
    </div>
  );
}
```

**Output**
```
// Initial render:
ProductCard rendered: Phone
ProductCard rendered: Watch

// Typing in search (only filter changes):
ProductCard rendered: Phone   ← only matching ones re-render

// Adding to cart (cart state changes, filtered + handleDelete don't):
(silence)   ← memo + stable refs block re-renders
```

---

[View Interview Questions](./interview.md)
