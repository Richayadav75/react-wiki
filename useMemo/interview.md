# useMemo Interview Questions

1. **What is memoization?**
   - An optimization technique where you cache the result of expensive function calls and return the cached result when the same inputs occur again.

2. **When should you NOT use useMemo?**
   - For cheap calculations. The overhead of checking dependencies and storing the value in memory might be greater than the cost of the calculation itself.

3. **Difference between useMemo and React.memo?**
   - `useMemo` memoizes a **value** inside a component. `React.memo` memoizes an entire **component** to prevent parent-driven re-renders.
