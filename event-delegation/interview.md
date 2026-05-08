# Event Delegation Interview Questions

1. **What is event delegation?**
   - A pattern where a single event listener is attached to a parent element to handle events from all its children. It relies on event bubbling — child events bubble up to the parent where the single handler catches them.

2. **What are the two main benefits of event delegation?**
   - (1) **Memory efficiency** — one listener instead of N listeners (one per child). (2) **Dynamic elements** — children added to the DOM after the listener is set up are automatically handled, because the listener is on the stable parent.

3. **What is the difference between event.target and event.currentTarget in delegation?**
   - `event.target` is the specific child that was clicked. `event.currentTarget` is always the parent element where the listener lives. In delegation you use `event.target` to identify which child triggered the event.

4. **Why use event.target.closest() instead of event.target directly?**
   - Users might click a child element inside the button (like an icon or span). `event.target` would then be that inner element, not the button. `.closest('.delete-btn')` traverses up from the clicked element to find the nearest matching ancestor — handling nested structures correctly.
   ```javascript
   // ❌ Fragile — breaks if user clicks the icon inside the button
   if (event.target.classList.contains('delete-btn')) { ... }

   // ✅ Robust
   const btn = event.target.closest('.delete-btn');
   if (btn) { ... }
   ```

5. **How do you use data-* attributes in event delegation?**
   - Store metadata on child elements using `data-*` attributes. The parent listener reads these from `event.target.closest(selector).dataset` to identify which child was interacted with.
   ```javascript
   // HTML: <li data-id="42" data-type="product">
   const item = event.target.closest('li');
   const { id, type } = item.dataset;
   ```

6. **Which events don't work well with delegation?**
   - `mouseenter` and `mouseleave` don't bubble — use `mouseover`/`mouseout` instead. Also be careful with events where `stopPropagation()` is called on a child, as the event won't reach the parent listener.

7. **Is event delegation always better than direct listeners?**
   - No. For a small number of stable elements, direct listeners are simpler and clearer. Delegation adds a condition check on every event, which could be wasteful for high-frequency events on a large DOM. Use delegation when the benefits (dynamic elements, large lists) justify it.

8. **How would you build a dynamic delete list with event delegation?**
   ```javascript
   document.querySelector('#list').addEventListener('click', (e) => {
     const btn = e.target.closest('.delete-btn');
     if (!btn) return;
     btn.closest('li').remove();
   });
   ```
   Items added later with `appendChild` are automatically handled.

9. **What is the advantage of delegation for dynamically added elements?**
   - The listener is on the parent, which is always in the DOM. New children don't need their own listeners — they just need to exist inside the parent for their events to bubble up to the shared handler.

10. **How does event delegation relate to performance in large lists?**
    - With 1000 list items and a listener on each, you have 1000 event listener objects in memory. With delegation, you have 1. The browser also has to track and manage each listener — fewer listeners mean less work during rendering and garbage collection.
