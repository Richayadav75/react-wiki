# useContext Interview Questions

1. **What problem does Context solve?**
   - Prop drilling: the process of passing props through multiple levels of components that don't need the data themselves.

2. **Does Context replace Redux?**
   - For many simple use cases, yes. However, Redux provides better debugging tools (DevTools), middleware support, and more optimized updates for very large state trees.

3. **What is a common pitfall of useContext?**
   - Every component consuming a context will re-render whenever the context value changes, even if they only use a small part of it.
