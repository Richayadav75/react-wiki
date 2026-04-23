- Category: Hooks
- Difficulty: Beginner
- Related: useReducer, useRef

`useState` is the most fundamental React hook. It lets you add **reactive state** to a function component — when the state value changes, React automatically re-renders the component.

## Syntax

```tsx
const [value, setValue] = useState<Type>(initialValue);
```

- `value` — the current state value (read-only in render)
- `setValue` — a setter function that schedules a re-render
- `initialValue` — the starting value (only used on the first render)

## Example

```tsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <button onClick={() => setCount(prev => prev - 1)}>Decrement</button>
    </div>
  );
}
```

## Functional updates

When the new state depends on the previous state, always use the **functional update** form to avoid stale closure bugs:

```tsx
// ❌ Potentially stale
setCount(count + 1);

// ✅ Always fresh
setCount(prev => prev + 1);
```

## Lazy initializer

If the initial state is expensive to compute, pass a **function** instead of a value — it runs only once:

```tsx
const [data, setData] = useState(() => expensiveComputation());
```

## Object state

React state setters replace, not merge. Spread the previous state to keep unchanged fields:

```tsx
const [form, setForm] = useState({ name: '', email: '' });

// ✅ Merge correctly
setForm(prev => ({ ...prev, name: 'Alice' }));
```

## Common Pitfalls

- **State is asynchronous** — reading `value` immediately after `setValue` still gives the old value within the same event handler.
- **Mutating state directly** (`value.push(x)`) won't trigger a re-render — always pass a new reference.
- **Unnecessary state** — if a value can be derived from props or other state, don't store it in state.
- **Missing the functional update form** — leads to stale closures in callbacks and effects.

## Practice
```tsx
function SearchInput() {
  const [query, setQuery] = useState("");
  
  return (
    <div className="p-4 border rounded shadow">
      <h3 className="font-bold mb-2">Search Case Study</h3>
      <input 
        type="text" 
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Type to search..."
        className="w-full p-2 border rounded"
      />
      <p className="mt-2 text-sm text-gray-600">
        Active Query: <strong>{query}</strong>
      </p>
    </div>
  );
}
```

## Interview Questions
1. **What happens if you mutate state directly instead of using the setter?**
   React will not detect the change because it checks for reference equality. No re-render will be triggered, and the UI will stay out of sync.
2. **Why does React batch state updates?**
   Batching reduces the number of re-renders, improving performance. Multiple state updates in the same event handler will only result in one render.

## Learn More
- [React Docs — useState](https://react.dev/reference/react/useState)
- [useState Deep Dive — Dan Abramov](https://overreacted.io/a-complete-guide-to-useeffect/)
