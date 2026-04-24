# Props vs State Interview Questions

1. **Can a component modify its own props?**
   - No, props are read-only (immutable) from the component's perspective.

2. **What is "Lifting State Up"?**
   - Moving state to the closest common ancestor of components that need to share that state.

3. **Difference between State and Props?**
   - State is managed within the component (private). Props are passed to the component (external) like function arguments.
