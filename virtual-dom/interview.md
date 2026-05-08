# Virtual DOM — Interview Questions

---

**1. What is the Virtual DOM and why does React use it?**

The Virtual DOM is a lightweight JavaScript object tree that mirrors the structure of the real browser DOM. React uses it because real DOM manipulation is slow — every change triggers style recalculation, layout (reflow), and repaint. By diffing in memory first and applying only the minimum necessary patches to the real DOM, React makes UI updates far more efficient.

```javascript
// JSX compiles to a Virtual DOM object (plain JS)
const element = <h1 className="title">Hello</h1>;

// Equivalent to:
const vdom = {
  type: "h1",
  props: { className: "title", children: "Hello" }
};
// This is just a JS object — no browser work yet
```

---

**2. Explain the three steps of React's update process.**

```text
Step 1 — Render:
  State or props change → component function runs → new Virtual DOM tree created

Step 2 — Diff:
  New VDOM compared with previous VDOM node-by-node (O(n) algorithm)
  → Produces a list of minimal change instructions (patches)

Step 3 — Commit / Patch:
  React applies only the patches to the real DOM
  Browser repaints only the changed elements
```

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  return <div><h1>Count: {count}</h1><button onClick={() => setCount(c => c + 1)}>+</button></div>;
}

// On click:
// Step 1: new VDOM created with "Count: 1"
// Step 2: diff finds h1 text changed "Count: 0" → "Count: 1"; button unchanged
// Step 3: only h1 text node updated in real DOM — button NOT touched
```

---

**3. What are the two key assumptions of React's diffing algorithm?**

```text
Assumption 1 — Different element types produce different trees:
  If <div> changes to <span>, React tears down the div tree entirely
  and builds the span tree from scratch (including remounting children).

Assumption 2 — Keys stabilize list item identity:
  Key tells React: "this list item with key='u1' is the same user across renders"
  Without key, React uses array index — causes wrong diffs when order changes.
```

```jsx
// Different type → full unmount/remount
// Old: <div><Child /></div>   New: <span><Child /></span>
// React: destroys div + Child, mounts fresh span + Child (Child loses state)

// Same type → update props only
// Old: <input type="text" value="hello" />
// New: <input type="text" value="world" />
// React: updates only the value attribute
```

---

**4. Why should you not use array index as a key in lists?**

Using index as a key breaks React's ability to identify which item is which when the list order changes. It leads to incorrect diffs, wasted renders, and bugs with component state.

```jsx
const items = [{ id: "a", text: "Apple" }, { id: "b", text: "Banana" }];

// WRONG — index as key
items.map((item, i) => <li key={i}>{item.text}</li>)
// Delete "Apple": Banana gets key=0, React updates Apple→Banana text (unnecessary update)

// CORRECT — stable unique ID as key
items.map(item => <li key={item.id}>{item.text}</li>)
// Delete "Apple": key="a" removed, key="b" untouched — 1 DOM operation
```

---

**5. What is Reconciliation?**

Reconciliation is React's process of syncing the Virtual DOM with the real DOM. After the diff identifies what changed, the commit phase (reconciliation) applies those changes to the browser's DOM.

```text
Reconciliation flow:
  render phase  → create new VDOM, diff against old VDOM (can be paused — Fiber)
  commit phase  → apply patches to real DOM (synchronous, cannot be interrupted)

Reconciliation is triggered by:
  → setState() / useState setter called
  → props change from parent
  → Context value changes
  → forceUpdate() (class components)
```

---

**6. What is React Fiber and how does it improve on the old reconciler?**

React Fiber (React 16+) makes the render phase incremental. Instead of processing the entire component tree in one blocking synchronous pass, Fiber splits work into small units and can pause, prioritize, and resume rendering.

```text
Old reconciler (React 15):
  Long render → blocks JavaScript thread → browser can't paint → UI freezes

Fiber (React 16+):
  Render work split into fiber units
  High priority (click/input) → yield to browser first → then resume low-priority work
  → Smooth animations, responsive input even during heavy renders
```

```jsx
// React 18 — useTransition exposes Fiber's priority scheduling
const [isPending, startTransition] = useTransition();

// Mark this update as low-priority — can be interrupted
startTransition(() => setSearchResults(filterItems(query)));

// High-priority update — happens immediately
setInputValue(e.target.value);
```

---

**7. Is the Virtual DOM always faster than direct DOM manipulation?**

Not always. For very small, simple updates, direct DOM manipulation (e.g., `element.textContent = "x"`) can be faster because it skips the VDOM overhead. React's advantage is at scale — when the UI is complex and many things might have changed. React minimizes the total real DOM work across a whole application, which is harder to do correctly by hand.

```text
Direct DOM manipulation (better for):
  Single, known element update: document.getElementById("price").textContent = "$99"
  Performance-critical animations (use refs/canvas instead)

Virtual DOM (better for):
  Complex component trees where you don't know what changed
  Many components reading from shared state
  Team code where manual DOM tracking is error-prone
```

---

**8. What is "batching" in React and how does it relate to the Virtual DOM?**

Batching is React grouping multiple state updates into a single re-render. Without batching, three `setState` calls would trigger three renders. With batching, they're all applied at once, producing one new VDOM, one diff, one patch.

```jsx
// React 18 — automatic batching (even in setTimeout and Promises)
function handleCheckout() {
  setCartOpen(false);   // \
  setLoading(true);     //  > batched together → ONE render, ONE diff, ONE DOM update
  setStep("payment");   // /
}

// React 17 — only batched inside event handlers
// In setTimeout: each setState would trigger a separate render

// React 18 with flushSync — opt out of batching when needed
import { flushSync } from "react-dom";
flushSync(() => setCartOpen(false));  // forces immediate render
setLoading(true); // separate render
```

---

**9. How do keys help with component state preservation?**

Keys tell React when to preserve a component's state and when to reset it. Same key = same component instance (state kept). Different key = React unmounts old, mounts new (state reset).

```jsx
// Use case: reset a form when the user switches tabs
function App() {
  const [activeTab, setActiveTab] = useState("profile");
  return (
    <>
      <button onClick={() => setActiveTab("profile")}>Profile</button>
      <button onClick={() => setActiveTab("settings")}>Settings</button>

      {/* key change forces form to reset its state when tab switches */}
      <Form key={activeTab} />
    </>
  );
}
// Without key: Form retains old input values when tab switches
// With key:    Form remounts fresh with empty inputs each time
```

---

**10. What is the difference between the render phase and the commit phase?**

```text
Render Phase:
  → React calls component functions, creates new VDOM, runs the diff
  → Can be interrupted and resumed (Fiber makes this possible)
  → No visible side effects — pure computation
  → useReducer and useState are processed here

Commit Phase:
  → React applies the diff to the real DOM
  → Runs useLayoutEffect (synchronously after DOM update)
  → Runs useEffect (asynchronously after browser has painted)
  → Cannot be interrupted — must complete atomically

Why this matters:
  → You should NEVER read/write the real DOM during the render phase
  → Use refs and useEffect for any real DOM interaction
```

```jsx
function Component() {
  // render phase — runs during VDOM creation (can be called multiple times in Concurrent Mode)
  const data = computeValue(); // must be pure

  useLayoutEffect(() => {
    // commit phase — synchronous, DOM is updated but browser hasn't painted yet
    // Use for measuring DOM dimensions
    const height = ref.current.getBoundingClientRect().height;
  });

  useEffect(() => {
    // after paint — safest place for subscriptions, fetch, timers
    const id = setInterval(tick, 1000);
    return () => clearInterval(id);
  }, []);
}
```
