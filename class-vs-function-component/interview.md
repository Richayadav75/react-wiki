# Class vs Function Component Interview Questions

1. **What is the main difference between Class and Functional components?**
   - **Class components** are ES6 classes that extend `React.Component` and use lifecycle methods.
   - **Functional components** are basic JavaScript functions that use Hooks to manage state and side effects.

2. **Why did React move towards Functional components?**
   - Functional components are more concise, easier to test, and don't require the confusing `this` keyword. They also allow for better logic sharing through Custom Hooks.

3. **Can you use state in a Functional component?**
   - Yes, since React 16.8, you can use the `useState` hook to manage local state in functional components.

4. **How do you handle lifecycle in Functional components?**
   - By using the `useEffect` hook, which can replicate the behavior of `componentDidMount`, `componentDidUpdate`, and `componentWillUnmount`.

5. **Is there any case where you MUST use a Class component?**
   - Yes, for **Error Boundaries**. Currently, there is no functional hook equivalent for catching errors in the component tree.

6. **What is the "constructor" equivalent in a Functional component?**
   - Functional components don't need a constructor. Initial state is set directly in the `useState` call, and other initialization can happen directly in the function body or a `useMemo` block.

7. **Which type of component is better for performance?**
   - Functional components are generally slightly more performant as they avoid the overhead of class instances and are easier for minifiers to optimize.
 Riverside.
 Riverside.
