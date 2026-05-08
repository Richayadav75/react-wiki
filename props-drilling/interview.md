# Prop Drilling Interview Questions

1. **What is Prop Drilling?**
   - It is the practice of passing props through several layers of components just to reach a deeply nested child component that needs the data, even though the intermediate components do not use it.

2. **Is Prop Drilling always bad?**
   - **No.** For small applications or components nested only 2-3 levels deep, prop drilling is simple and explicit. It only becomes a problem when it makes the code hard to maintain or understand.

3. **What are the main disadvantages of Prop Drilling?**
   - It leads to "boilerplate" code in intermediate components.
   - It makes refactoring difficult (if you change a prop name, you must update multiple files).
   - It makes it harder to reuse intermediate components independently.

4. **How can you avoid Prop Drilling without using Context?**
   - **Component Composition**: Pass the child component itself as a prop (`children`). This allows the parent to give data directly to the child while the "middleman" component just renders whatever it is given.

5. **What is the Context API?**
   - It is a built-in React feature that allows you to share data "globally" with an entire tree of components without passing it through props manually at every level.

6. **When should you use Redux instead of Context to solve Prop Drilling?**
   - Use Redux for large-scale applications with complex state transitions, high-frequency updates, or when you need powerful debugging tools (DevTools). Use Context for low-frequency updates like Theme, Language, or Auth state.

7. **Does Prop Drilling affect performance?**
   - Not significantly by itself. However, because the intermediate components must re-render whenever the props change (even if they don't use them), it can lead to unnecessary re-renders in very large trees.
 Riverside.
 Riverside.
