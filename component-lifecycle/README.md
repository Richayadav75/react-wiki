- Category: Fundamentals
- Difficulty: Beginner
- Related: useEffect, useState

React function components don't have explicit lifecycle methods like class components, but they go through the same phases: **Mount → Update → Unmount**. Understanding how these map to hooks is key to avoiding bugs.

## The three phases

### Mount (component appears in the DOM)

```tsx
useEffect(() => {
  // Runs once after first render (componentDidMount equivalent)
  console.log('mounted');
}, []);
```

### Update (props or state changes)

```tsx
useEffect(() => {
  // Runs after every render where `value` changed (componentDidUpdate equivalent)
  console.log('value changed to', value);
}, [value]);
```

### Unmount (component removed from the DOM)

```tsx
useEffect(() => {
  const subscription = subscribe();
  return () => {
    // Cleanup — runs before unmount (componentWillUnmount equivalent)
    subscription.unsubscribe();
  };
}, []);
```

## Lifecycle timeline

```
Mount:   render → paint → useEffect ([] deps)
Update:  render → paint → useEffect cleanup → useEffect run
Unmount: useEffect cleanup → DOM removed
```

## Class ↔ Hook equivalents

| Class Lifecycle | Hook Equivalent |
|----------------|-----------------|
| `componentDidMount` | `useEffect(() => {}, [])` |
| `componentDidUpdate` | `useEffect(() => {}, [dep])` |
| `componentWillUnmount` | `useEffect(() => { return cleanup }, [])` |
| `shouldComponentUpdate` | `React.memo` / `useMemo` / `useCallback` |
| `getDerivedStateFromProps` | Compute during render (no hook needed) |

## useLayoutEffect

`useLayoutEffect` fires **synchronously after DOM mutations but before the browser paints** — useful for measuring DOM nodes or synchronizing animations.

```tsx
useLayoutEffect(() => {
  // Runs before paint — safe to read layout
  const rect = ref.current.getBoundingClientRect();
  setHeight(rect.height);
}, []);
```

> Prefer `useEffect` by default; use `useLayoutEffect` only when you need synchronous DOM measurement.

## Common Pitfalls

- **Relying on component order** — React may unmount and remount components (Strict Mode does this intentionally in dev).
- **Skipping cleanup** — Always clean up subscriptions, timers, and listeners in the return function.
- **Running effects before mount** — A ref assigned in JSX (`ref={myRef}`) is only populated *after* render, not before.

## Learn More

- [React Docs — useEffect](https://react.dev/reference/react/useEffect)
- [React Docs — useLayoutEffect](https://react.dev/reference/react/useLayoutEffect)
- [React Lifecycle Diagram](https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/)

---

[View Interview Questions](./interview.md)
