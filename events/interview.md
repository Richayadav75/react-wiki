# Events Interview Questions

1. **What is Event Bubbling?**
   - The process where an event starts from the target element and "bubbles up" through its ancestors in the DOM tree.

2. **How do you stop an event from propagating?**
   - By calling `event.stopPropagation()`.

3. **Difference between event.target and event.currentTarget?**
   - `event.target` is the actual element that triggered the event. `event.currentTarget` is the element that the event listener is attached to.
