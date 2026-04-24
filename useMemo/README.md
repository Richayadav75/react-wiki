- Category: Performance
- Difficulty: Intermediate
- Related: useCallback

### What is useMemo?
`useMemo` is a performance hook that "memoizes" (caches) the result of an expensive calculation.

---

### 1. Performance Optimization
**Theory**: It only re-calculates the value when one of its dependencies changes. This prevents heavy work on every single render.

**Key Features**:
- **Caching**: Saves the result in memory.
- **Efficiency**: Skips unnecessary math/logic.

**Example**:
```tsx
const expensiveResult = useMemo(() => {
  return heavyCalculation(data);
}, [data]); // Only recalculates if 'data' changes
```
