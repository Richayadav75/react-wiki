# React.memo Interview Questions

1. **What is React.memo and what problem does it solve?**
   - `React.memo` is a higher-order component that prevents a functional component from re-rendering when its parent re-renders but its props haven't changed. It solves the problem of wasted renders in child components that receive the same props every time.

2. **What kind of comparison does React.memo use by default?**
   - Shallow comparison. It checks if each prop value is the same reference (for objects/functions) or the same value (for primitives). Two separate `{ color: 'red' }` objects fail the check even though they look equal.

3. **Why does React.memo fail when you pass inline objects or functions as props?**
   - Because `{}` creates a new object reference on every render. Shallow comparison sees `oldProp !== newProp` and allows the re-render. Fix: use `useMemo` for objects and `useCallback` for functions to stabilize references.
   ```jsx
   // ❌ Breaks memo
   <Child style={{ color: 'red' }} onClick={() => handle()} />

   // ✅ Stable references
   const style   = useMemo(() => ({ color: 'red' }), []);
   const onClick  = useCallback(() => handle(), []);
   <Child style={style} onClick={onClick} />
   ```

4. **How do you provide a custom comparison to React.memo?**
   - Pass a second argument: a function `(prevProps, nextProps) => boolean`. Return `true` to skip re-render, `false` to allow it.
   ```jsx
   const Card = memo(Component, (prev, next) => prev.id === next.id);
   ```

5. **Does React.memo prevent re-renders caused by useContext changes?**
   - No. Any component that calls `useContext` re-renders when the context value changes, regardless of `React.memo`. Memo only guards against prop changes.

6. **What is the relationship between React.memo, useCallback, and useMemo?**
   - They work together: `React.memo` guards the child, `useCallback` stabilizes function props, `useMemo` stabilizes object/array props. Using only one without the others often leaves performance gains on the table.

7. **When should you NOT use React.memo?**
   - When the component renders cheaply (no complex JSX or calculations), when props change on every render anyway, or when you haven't profiled and confirmed a real performance problem. The comparison overhead can exceed the render cost for simple components.

8. **Does React.memo work with class components?**
   - No. For class components, use `PureComponent` (shallow comparison) or implement `shouldComponentUpdate` for custom logic.

9. **How is React.memo different from PureComponent?**
   - `React.memo` is for functional components; `PureComponent` is for class components. Both do shallow prop comparison. `PureComponent` also shallowly compares state.

10. **How do you verify React.memo is working correctly?**
    - Add a `console.log` inside the component. If memo is working, the log should not fire when unrelated parent state changes. Use React DevTools Profiler to see re-render highlights — grey means memo blocked it.
