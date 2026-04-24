# Event Delegation Interview Questions

1. **What is Event Delegation?**
   - Attaching a single event listener to a parent element instead of multiple listeners to its children, leveraging event bubbling.

2. **Why use event delegation?**
   - Memory efficiency (one listener vs many) and handling dynamically added child elements.

3. **How do you identify which child was clicked in delegation?**
   - By checking the `event.target` property inside the parent's listener.
