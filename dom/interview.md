# DOM Interview Questions

1. **Difference between innerHTML and textContent?**
   - `innerHTML` parses the string as HTML. `textContent` treats it as raw text, which is safer against XSS (Cross-Site Scripting) attacks.

2. **What is the difference between HTMLCollection and NodeList?**
   - `HTMLCollection` (e.g., from `getElementsByTagName`) is live (updates with DOM changes). `NodeList` (e.g., from `querySelectorAll`) is usually static.

3. **What is a "DocumentFragment"?**
   - A lightweight, "off-screen" DOM node used to group elements before appending them to the real DOM, reducing reflows and repaints.
