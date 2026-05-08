# Error Boundaries Interview Questions

1. **What is an Error Boundary in React?**
   - It is a class component that catches JavaScript errors anywhere in its child component tree and displays a fallback UI instead of crashing the whole application.

2. **Why must Error Boundaries be class components?**
   - Because they rely on specific lifecycle methods (`getDerivedStateFromError` and `componentDidCatch`) that do not have functional hook equivalents yet.

3. **What are the two lifecycle methods used in Error Boundaries?**
   - `static getDerivedStateFromError(error)`: Used to update state and show a fallback UI.
   - `componentDidCatch(error, info)`: Used for logging the error to an external service.

4. **Name three things an Error Boundary cannot catch.**
   1. Errors in Event Handlers.
   2. Asynchronous code (e.g., `setTimeout`).
   3. Server-side rendering.
   4. Errors in the Error Boundary itself.

5. **How do you handle errors in an Event Handler if Error Boundaries can't?**
   - Use a standard `try...catch` block inside the event handler function.

6. **Should you wrap your entire app in a single Error Boundary?**
   - Generally, no. It's better to use them "locally" around complex or unstable features (like third-party widgets or complex charts) so that a failure in one part of the UI doesn't break the entire page.

7. **What is the benefit of using Error Boundaries?**
   - They provide a much better user experience by preventing "white screens of death" and allowing the user to continue using other parts of the application even if one section fails.
 Riverside.
 Riverside.
