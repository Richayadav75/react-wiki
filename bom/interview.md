# BOM Interview Questions

1. **What is the root object of the BOM?**
   - The `window` object. It represents the browser window and is the global object for all client-side JavaScript.

2. **How can you redirect a user to a different URL using JavaScript?**
   - By setting `window.location.href = "https://newurl.com"` or using `window.location.replace("https://newurl.com")`.

3. **What is the difference between `window.location.assign()` and `window.location.replace()`?**
   - `assign()`: Loads a new document and adds it to the session history (allows the user to go back).
   - `replace()`: Loads a new document but replaces the current entry in the session history (user cannot go back).

4. **How do you find the width and height of the browser's viewport?**
   - Use `window.innerWidth` and `window.innerHeight`.

5. **What is the purpose of `window.navigator.userAgent`?**
   - It is a string that identifies the user's browser, version, and operating system. It is commonly used for "feature detection" or browser-specific fixes.

6. **How can you check if the user's browser is currently online?**
   - Use the `window.navigator.onLine` property, which returns a boolean.

7. **What is the difference between the BOM and the DOM?**
   - The **DOM** is for manipulating the HTML document structure and content.
   - The **BOM** is for interacting with the browser environment (e.g., URL, screen size, history, storage).
 Riverside.
 Riverside.
