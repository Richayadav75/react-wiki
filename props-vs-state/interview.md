# Props vs State Interview Questions

1. **What is the main difference between Props and State?**
   - **Props** are passed into a component from its parent (like function arguments) and are immutable.
   - **State** is managed internally by the component (like local variables) and can change over time via a setter function.

2. **Can a component modify its own props?**
   - **No.** Props are read-only. Modifying props would break React's "one-way data flow" and lead to unpredictable UI bugs.

3. **What happens when a component's state changes?**
   - React schedules a re-render for that component and all of its children. It compares the Virtual DOM to the real DOM and updates only what is necessary.

4. **What is "Lifting State Up"?**
   - It is the practice of moving state from a child component to its parent so that multiple children can share and stay in sync with the same data.

5. **What is "Prop Drilling"?**
   - It is the process of passing data through multiple levels of components (middlemen) that don't actually need the data, just to reach a deeply nested child.

6. **How do you pass data from a child to a parent?**
   - You pass a **callback function** from the parent to the child as a prop. The child then calls that function with the data it wants to send back.

7. **Is state shared between multiple instances of the same component?**
   - **No.** Each instance of a component maintains its own independent state. If you have two `<Counter />` components, clicking one will not update the other.
 Riverside.
 Riverside.
