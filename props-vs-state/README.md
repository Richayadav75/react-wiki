- Category: Fundamentals
- Difficulty: Beginner
- Related: useState, useContext

**Props** and **state** are both inputs that affect what a component renders, but they serve fundamentally different roles. Confusing the two is one of the most common beginner mistakes.

## Props — data flows down

Props are **read-only** values passed from a parent component. A component cannot change its own props.

```tsx
// Parent passes data down
function App() {
  return <Greeting name="Alice" age={28} />;
}

// Child reads, never writes
function Greeting({ name, age }: { name: string; age: number }) {
  return <p>Hello, {name}! You are {age} years old.</p>;
}
```

Props can be anything: strings, numbers, booleans, objects, arrays, functions, and even JSX (via `children`).

## State — local reactive memory

State is **private** to the component that owns it. Changing state schedules a re-render.

```tsx
function Toggle() {
  const [isOn, setIsOn] = useState(false);

  return (
    <button onClick={() => setIsOn(prev => !prev)}>
      {isOn ? 'ON' : 'OFF'}
    </button>
  );
}
```

## Side-by-side comparison

| | Props | State |
|---|---|---|
| Who controls it? | Parent component | The component itself |
| Mutable by the component? | ❌ No | ✅ Yes |
| Causes re-render when changes? | ✅ Yes (parent re-renders) | ✅ Yes |
| Accessible to children? | Via passing down | Via passing as prop |
| Initial source | Parent | `useState` initializer |

## Lifting state up

When two sibling components need to share state, **lift it** to their closest common parent:

```tsx
function Parent() {
  const [count, setCount] = useState(0);

  return (
    <>
      <Display count={count} />          {/* prop */}
      <Counter onIncrement={() => setCount(c => c + 1)} /> {/* callback prop */}
    </>
  );
}
```

## Derived values — don't store what you can compute

If a value can be derived from props or state, **compute it during render** instead of storing it in state:

```tsx
// ❌ Redundant state
const [fullName, setFullName] = useState(`${firstName} ${lastName}`);

// ✅ Derived value
const fullName = `${firstName} ${lastName}`;
```

## Common Pitfalls

- **Trying to modify props** — This causes TypeScript errors and breaks React's data flow.
- **Copying props into state** — Creates a stale copy that diverges from the source of truth.
- **Over-lifting state** — Lift state only as high as needed; over-lifting causes unnecessary re-renders.

## Learn More

- [React Docs — Passing Props to a Component](https://react.dev/learn/passing-props-to-a-component)
- [React Docs — State: A Component's Memory](https://react.dev/learn/state-a-components-memory)
- [Thinking in React](https://react.dev/learn/thinking-in-react)
