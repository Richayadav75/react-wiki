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

## Learn More

- [React Docs — useState](https://react.dev/reference/react/useState)
- [useState Deep Dive — Dan Abramov](https://overreacted.io/a-complete-guide-to-useeffect/)
