- Category: DOM Manipulation
- Difficulty: Intermediate
- Related: Event Loop

# Event Delegation

Event delegation is a technique where you attach a single event listener to a parent element to manage all events for its children, instead of attaching listeners to every individual child.

## How it works
It relies on **Event Bubbling**: when an event happens on an element, it first runs the handlers on it, then on its parent, then all the way up on other ancestors.

## Example

```javascript
document.querySelector('#parent-list').addEventListener('click', (event) => {
  if (event.target.tagName === 'LI') {
    console.log('List item clicked:', event.target.textContent);
  }
});
```

## Benefits
1. **Memory efficiency**: One listener instead of hundreds.
2. **Dynamic elements**: Works for children added to the DOM *after* the listener is attached.

## Practice: Custom Tab Switcher
Using delegation to handle multiple button clicks.

```javascript
const tabContainer = document.querySelector('.tabs');

tabContainer.addEventListener('click', (e) => {
  const tabButton = e.target.closest('.tab-btn');
  if (!tabButton) return;
  
  // Logic to switch tabs
  console.log('Switching to tab:', tabButton.dataset.tab);
});
```

## Interview Questions
1. **What is the difference between Bubbling and Capturing?**
   Bubbling goes from target up to root. Capturing goes from root down to target. Most handlers use bubbling by default.
2. **How do you stop an event from bubbling?**
   Use `event.stopPropagation()`.
3. **What is `event.target` vs `event.currentTarget`?**
   `event.target` is the element that triggered the event (the child). `event.currentTarget` is the element the listener is attached to (the parent).

---

[View Interview Questions](./interview.md)
