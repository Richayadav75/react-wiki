# API Calls in React Interview Questions

1. **Where should you make an API call in a functional component?**
   - Inside the `useEffect` hook. This ensures the call happens after the component has rendered.

2. **Why is the dependency array in `useEffect` important for API calls?**
   - It controls when the fetch happens. An empty array `[]` means it runs only once on mount. If it contains a variable like `[userId]`, the fetch will re-run every time that variable changes.

3. **What is the "Race Condition" in data fetching?**
   - It happens when two requests are made quickly, and the older one arrives *after* the newer one, overwriting the correct data with stale data. This is solved using a "cleanup" variable or an `AbortController`.

4. **How do you handle loading states in React?**
   - By using a piece of state (e.g., `const [isLoading, setIsLoading] = useState(true)`) and conditionally rendering a spinner or loading text while the request is pending.

5. **What are the benefits of using Axios over the Fetch API?**
   - Axios automatically transforms JSON data, supports request/response interceptors, and has better error handling (it rejects on 4xx/5xx codes, whereas Fetch only rejects on network failure).

6. **What is React Query (TanStack Query)?**
   - It is a powerful data-fetching library that handles complex tasks like caching, background re-fetching, and synchronization with the server state automatically.

7. **How do you handle API errors in React?**
   - Use a `try...catch` block (with async/await) or `.catch()` (with promises) to update an `error` state. You can then render an error message if that state is populated.
 Riverside.
 Riverside.
