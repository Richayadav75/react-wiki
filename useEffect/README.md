- Category: Hooks
- Difficulty: Beginner
- Related: useState

### What is useEffect?
`useEffect` allows you to perform "side effects" in your components, such as data fetching, subscriptions, or manually changing the DOM.

---

### 1. Component Lifecycle
**Theory**: It runs after the component renders. It can be configured to run once, every time, or only when certain data changes.

**Working Flow**
```text
[ Render ] --> [ Run Effect ] --> ( Cleanup on Unmount )
```

**Key Features**:
- **Dependency Array**: Controls when the effect runs (`[]` = once, `[val]` = when val changes).
- **Cleanup Function**: Used to stop timers or unsubscribe from APIs.

**Example (Fetching Data)**:
```tsx
useEffect(() => {
  console.log("Component has mounted!");
  
  return () => {
    console.log("Component will unmount (Cleanup)!");
  };
}, []); // Empty array means run only once
```
