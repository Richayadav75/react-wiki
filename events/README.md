- Category: DOM Manipulation
- Difficulty: Beginner
- Related: dom, event-delegation, bom

### Events — Responding to User and Browser Actions
Events are signals fired by the browser when something happens — a user clicks a button, presses a key, submits a form, or the page finishes loading. JavaScript listens for these signals and runs a function in response (an event handler/listener).

**Analogy**
A hotel's front desk. The desk (browser) monitors everything happening in the hotel. When a guest presses the call button (event), the desk rings up the assigned staff member (event listener) who then handles the request (callback function). The desk keeps listening for the next call without blocking anything else.

---

### 1. addEventListener — The Modern Way to Attach Events

**Theory**
`element.addEventListener(eventType, handlerFn, options)` attaches a function to be called when a specific event fires on that element. It is the preferred modern approach — it allows multiple listeners on the same event and is fully controllable (can be removed).

**Working Flow**

![flow-chart](flow-chart.png)

**Example**
```javascript
// Step 1 — Select the element
const button = document.querySelector('#myBtn');

// Step 2 — Define the handler
function handleClick(event) {
  console.log('Button clicked!');
  console.log('Clicked element:', event.target.id);
}

// Step 3 — Attach the listener
button.addEventListener('click', handleClick);

// Multiple listeners on same element — all fire
button.addEventListener('click', () => console.log('Second listener'));
```

**Output**
```
// Click the button:
Button clicked!
Clicked element: myBtn
Second listener
```

**Explanation**
Unlike the old `onclick = fn` (which overwrites), `addEventListener` stacks listeners. Both functions run. The `event` object is automatically passed by the browser and contains all info about what happened.

---

### 2. The Event Object — What's Inside

**Theory**
Every event handler receives an `event` object (often named `e` or `event`) with properties describing the event: what type it was, what element triggered it, mouse position, which key was pressed, etc.

**Working Flow**

![flow-chart-2](flow-chart-2.png)

**Example**
```javascript
document.addEventListener('click', function(event) {
  console.log('type:',          event.type);          // "click"
  console.log('target:',        event.target);         // element clicked
  console.log('currentTarget:', event.currentTarget);  // element with listener
  console.log('clientX/Y:',    event.clientX, event.clientY); // mouse position
  console.log('timeStamp:',    event.timeStamp);       // ms since page load
});

// Key events
document.addEventListener('keydown', function(e) {
  console.log('key:',     e.key);       // "Enter", "a", "Escape"
  console.log('code:',    e.code);      // "KeyA", "Enter"
  console.log('ctrlKey:', e.ctrlKey);   // true if Ctrl held
  console.log('shiftKey:', e.shiftKey); // true if Shift held
});
```

**Output**
```
// Click at position (200, 350):
type:          "click"
target:        <div id="box">
currentTarget: document
clientX/Y:     200 350

// Press Ctrl+S:
key:      "s"
code:     "KeyS"
ctrlKey:  true
shiftKey: false
```

---

### 3. Event Bubbling and Capturing

**Theory**
When you click a nested element, the event travels in two phases:
1. **Capture phase** — from `window` down to the target (rarely used)
2. **Bubble phase** — from target back up to `window` (default)

Most handlers run in the bubble phase. `stopPropagation()` stops the event from travelling further up the chain.

**Working Flow**

![flow-chart-3](flow-chart-3.png)

**Example**
```javascript
const outer = document.querySelector('.outer');
const inner = document.querySelector('.inner');

outer.addEventListener('click', () => console.log('Outer clicked'));
inner.addEventListener('click', () => console.log('Inner clicked'));

// When you click .inner:
// "Inner clicked"
// "Outer clicked"   ← event bubbles up

// Stop bubbling
inner.addEventListener('click', (e) => {
  e.stopPropagation();
  console.log('Inner — bubble stopped');
});
// Now clicking .inner logs only: "Inner — bubble stopped"

// Capture phase — third argument true
outer.addEventListener(
  'click',
  () => console.log('Outer captured (before inner)'),
  true  // ← capture phase
);
```

**Output**
```
// Click .inner (no stopPropagation):
Inner clicked
Outer clicked

// Click .inner (with stopPropagation):
Inner — bubble stopped
(Outer does NOT fire)

// Click .inner (with capture listener):
Outer captured (before inner)   ← fires first
Inner clicked
Outer clicked
```

---

### 4. preventDefault — Blocking Default Browser Behaviour

**Theory**
Many elements have built-in browser behaviour: links navigate, form submits reload the page, right-click opens context menu. `event.preventDefault()` cancels this default action so you can handle it yourself.

**Working Flow**

![flow-chart-4](flow-chart-4.png)

**Example**
```javascript
// Prevent form reload on submit
const form = document.querySelector('form');
form.addEventListener('submit', function(event) {
  event.preventDefault();   // stops page reload

  const data = new FormData(form);
  console.log('Name:', data.get('name'));
  console.log('Email:', data.get('email'));
  // Now handle with fetch / AJAX
});

// Prevent link navigation
document.querySelector('a').addEventListener('click', function(event) {
  event.preventDefault();
  console.log('Link click intercepted:', event.target.href);
  // Do something else (SPA navigation, modal, etc.)
});

// Prevent right-click context menu
document.addEventListener('contextmenu', function(event) {
  event.preventDefault();
  console.log('Custom context menu at:', event.clientX, event.clientY);
});
```

**Output**
```
// Form submitted:
Name:  Alice
Email: alice@example.com
(page does NOT reload)

// Link clicked:
Link click intercepted: https://example.com
(browser does NOT navigate)
```

---

### 5. Common Event Types

**Theory**
Events cover everything from mouse movements to keyboard input to network changes. Knowing which event to use for each scenario is fundamental.

**Working Flow**

![flow-chart-5](flow-chart-5.png)

**Example**
```javascript
const input  = document.querySelector('input');
const box    = document.querySelector('.box');

// Mouse events
box.addEventListener('click',      e => console.log('click'));
box.addEventListener('dblclick',   e => console.log('dblclick'));
box.addEventListener('mouseenter', e => console.log('mouse entered'));
box.addEventListener('mouseleave', e => console.log('mouse left'));
box.addEventListener('mousemove',  e => console.log(`at ${e.clientX},${e.clientY}`));

// Keyboard events (on document or input)
input.addEventListener('keydown',  e => console.log('key down:', e.key));
input.addEventListener('keyup',    e => console.log('key up:', e.key));

// Form / input events
input.addEventListener('input',    e => console.log('value:', e.target.value));
input.addEventListener('change',   e => console.log('committed:', e.target.value));
input.addEventListener('focus',    () => console.log('focused'));
input.addEventListener('blur',     () => console.log('blurred'));

// Window / document events
window.addEventListener('resize',  () => console.log('window resized'));
window.addEventListener('scroll',  () => console.log('scrolled'));
document.addEventListener('DOMContentLoaded', () => console.log('DOM ready'));
window.addEventListener('load',    () => console.log('everything loaded'));
```

---

### 6. Removing Listeners — Cleanup and once Option

**Theory**
Event listeners accumulate if not removed — a common source of memory leaks. Remove a listener by passing the exact same function reference to `removeEventListener`. Alternatively, use the `{ once: true }` option to auto-remove after first fire.

**Working Flow**

![flow-chart-6](flow-chart-6.png)

**Example**
```javascript
const button = document.querySelector('button');

// Named function — can be removed
function handleClick() {
  console.log('Clicked!');
}

button.addEventListener('click', handleClick);

// Later — remove the exact same reference
button.removeEventListener('click', handleClick);
// Button clicks no longer trigger handleClick

// ❌ This does NOT work — anonymous function creates new reference each time
button.addEventListener('click', () => console.log('Hi'));
button.removeEventListener('click', () => console.log('Hi')); // no-op

// { once: true } — fires once, then auto-removes itself
button.addEventListener('click', () => {
  console.log('I only fire once');
}, { once: true });

// AbortController — remove multiple listeners at once
const controller = new AbortController();
const { signal } = controller;

window.addEventListener('resize', handleResize, { signal });
window.addEventListener('scroll', handleScroll, { signal });
document.addEventListener('keydown', handleKey, { signal });

// Remove all three with one call
controller.abort();
```

**Output**
```
// After removeEventListener:
Button click → (silence)

// { once: true } — first click:
I only fire once

// Second click:
(silence — listener was removed)

// After controller.abort():
Resize, scroll, keydown → all removed simultaneously
```

---

[View Interview Questions](./interview.md)
