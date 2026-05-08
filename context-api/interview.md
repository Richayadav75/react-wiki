# Context API Interview Questions

1. **What is the Context API?**
   - It is a React feature that allows you to share data through the component tree without having to pass props down manually at every level. It solves the problem of "Prop Drilling".

2. **What are the three main components of the Context API?**
   1. `createContext`: To create the context.
   2. `Provider`: To provide the value to the children.
   3. `useContext` (or Consumer): To consume the value inside a component.

3. **When should you use the Context API instead of Props?**
   - Use Context for "global" data that many components across different levels of the tree need (e.g., Theme, User Authentication, or Language settings).

4. **Does Context replace Redux?**
   - For many small to medium apps, yes. However, for large-scale applications with complex state transitions, high-frequency updates, and the need for powerful debugging tools, Redux is still preferred.

5. **What is the performance drawback of the Context API?**
   - Every time the context value changes, all components that consume that context are re-rendered. This can lead to performance issues if the context holds large amounts of data or updates very frequently.

6. **Can you have multiple Providers in one app?**
   - Yes! You can nest multiple providers (e.g., one for Theme, one for Auth). Components will always consume the value from the **closest** provider above them in the tree.

7. **How do you update a Context value from a child component?**
   - You pass a **setter function** (from `useState`) along with the data inside the context's `value` object. The child can then call that function to update the global state.
 Riverside.
 Riverside.
