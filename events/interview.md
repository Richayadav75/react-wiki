# Events Interview Questions

1. **What is an event in JavaScript?**
   - A signal fired by the browser when something happens — user clicks, keypresses, form submissions, page load, window resize. JavaScript listens for these signals and runs a callback function in response.

2. **What is the difference between addEventListener and onclick?**
   - `onclick = fn` only allows one handler — assigning a second one overwrites the first. `addEventListener` stacks handlers — multiple functions can listen to the same event on the same element. `addEventListener` also supports options like `{ once: true }` and `{ capture: true }`.

3. **What is event bubbling?**
   - After an event fires on an element, it travels up through ancestor elements, triggering any matching listeners on each parent. Clicking a `<button>` inside a `<div>` triggers both the button's and the div's click listeners.

4. **What is the difference between event.target and event.currentTarget?**
   - `event.target` is the innermost element that triggered the event (what was actually clicked). `event.currentTarget` is the element that the listener is attached to. They differ when bubbling is involved.

5. **How do you stop an event from bubbling?**
   - Call `event.stopPropagation()`. Use sparingly — it can break event delegation patterns on parent elements.

6. **What does event.preventDefault() do?**
   - Cancels the browser's default behaviour for that event. For a form submit it prevents page reload, for a link it prevents navigation, for a context menu it prevents the menu from appearing.

7. **What is the difference between the `input` and `change` events on an input?**
   - `input` fires on every keystroke as the value changes. `change` fires only when the user finishes editing and moves focus away (commits the change). For real-time feedback use `input`; for final value use `change`.

8. **What are the options object parameters for addEventListener?**
   - `{ capture: true }` — listen in the capture phase instead of bubble. `{ once: true }` — auto-remove the listener after it fires once. `{ passive: true }` — signals the handler won't call `preventDefault`, allowing the browser to optimize scroll performance.

9. **How do you remove an event listener?**
   - Pass the exact same function reference to `removeEventListener`. Anonymous functions cannot be removed this way — always store the handler in a variable if you plan to remove it.
   ```javascript
   function handler() { ... }
   el.addEventListener('click', handler);
   el.removeEventListener('click', handler); // works
   ```

10. **What is the difference between event capturing and event bubbling?**
    - **Capturing** (trickling) goes from `window` down to the target. **Bubbling** goes from the target back up to `window`. Most handlers use bubbling (default). Use `addEventListener(type, fn, true)` for capture phase.
