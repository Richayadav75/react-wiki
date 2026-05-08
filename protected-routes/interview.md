# Protected Routes Interview Questions

1. **What is a Protected Route?**
   - It is a route that can only be accessed by authenticated users. If a user is not logged in, the application redirects them to a login or signup page.

2. **How do you implement a Protected Route in React Router?**
   - By creating a wrapper component (e.g., `ProtectedRoute`) that checks the user's authentication status. It either renders the protected component or uses `<Navigate />` to redirect the user.

3. **What is the purpose of the `<Outlet />` component in a Protected Route?**
   - It is used in React Router to render the nested child routes of the `ProtectedRoute` wrapper. This allows you to wrap multiple routes in a single protection logic block.

4. **Is client-side routing alone enough for security?**
   - **No.** Client-side routing is only for UX (User Experience). It hides the UI from unauthorized users. Real security must happen on the **Server-side** (API) by verifying the authentication token for every sensitive data request.

5. **How do you persist a user's login state?**
   - By storing an authentication token (like JWT) in `localStorage`, `sessionStorage`, or a secure `httpOnly` cookie. When the app loads, you check for this token to re-authenticate the user.

6. **What is the "replace" prop in the `<Navigate />` component used for?**
   - It replaces the current entry in the browser history stack with the new one. This is useful during redirects so that if the user clicks the "Back" button, they don't get stuck in a redirect loop back to the login page.

7. **How do you handle "Unauthorized" (403) errors after a user is logged in?**
   - Even if a user is logged in, they might not have permission to see a specific resource. You should handle this by checking their **roles** (e.g., Admin vs User) in your routing logic or by responding to 403 errors from your API with a "Forbidden" page.
 Riverside.
 Riverside.
