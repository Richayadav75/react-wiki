- Category: Performance
- Difficulty: Intermediate
- Related: useMemo, useCallback

# React.memo

`React.memo` is a higher-order component that wraps a functional component to prevent unnecessary re-renders if its props haven't changed.

## How it works
React normally re-renders a child component whenever its parent re-renders. `React.memo` changes this behavior by performing a **shallow comparison** of the component's props.

## Example

```tsx
import React from 'react';

const ExpensiveComponent = React.memo(({ data }) => {
  console.log('Rendering expensive component...');
  return <div>{data}</div>;
});
```

## Custom Comparison
You can provide a second argument to `React.memo` to customize how props are compared.

```tsx
export default React.memo(MyComponent, (prevProps, nextProps) => {
  return prevProps.id === nextProps.id; // Only re-render if ID changes
});
```

## Practice: Optimization Case Study
When using `React.memo`, be careful with object/function props. Use `useCallback` or `useMemo` in the parent to ensure prop references stay stable.

```tsx
function Parent() {
  const [count, setCount] = useState(0);
  
  // Use useCallback so the function reference doesn't change on every render
  const handleClick = useCallback(() => {
    console.log('Clicked');
  }, []);

  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
      <MemoizedChild onClick={handleClick} />
    </>
  );
}
```

## Interview Questions
1. **Does `React.memo` stop re-renders caused by state changes inside the component?**
   No. It only skips re-renders caused by parent updates if props are identical. State or Context changes inside the component still trigger a re-render.
2. **When should you NOT use `React.memo`?**
   Don't use it for cheap-to-render components. The overhead of the shallow comparison might be more expensive than the render itself.
3. **What kind of comparison does it do by default?**
   A shallow comparison of props.
Ref references or objects passed as props must be stabilized to work effectively with `React.memo`.

---

[View Interview Questions](./interview.md)
