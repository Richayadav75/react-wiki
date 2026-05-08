- Category: React Hooks
- Difficulty: Intermediate
- Related: useState, useEffect, hooks

### useRef — A Mutable Box That Doesn't Trigger Re-renders
`useRef` returns a mutable ref object whose `.current` property is initialized with the value you pass. The key characteristic: **changing `.current` does not trigger a re-render**. It persists the value across renders like state, but silently — React doesn't know or care when you change it.

**Analogy**
A sticky note on your monitor. You can write on it and erase it any time — React doesn't know it exists, so the UI never updates because of it. But the note is always there when you look at it. Unlike a whiteboard (state) that redraws the whole room every time you write on it.

---

### 1. DOM References — Accessing Elements Directly

**Theory**
The most common use of `useRef` is attaching it to a JSX element via the `ref` attribute. React then sets `ref.current` to the DOM node after render, letting you call native DOM methods (focus, scrollIntoView, play, etc.) directly.

**Working Flow**

![flow-chart](flow-chart.png)

**Example**
```jsx
import { useRef } from 'react';

function SearchBar() {
  const inputRef = useRef(null);

  // Focus the input programmatically
  function handleFocus() {
    inputRef.current.focus();
  }

  // Select all text inside input
  function handleSelect() {
    inputRef.current.select();
  }

  return (
    <div>
      <input ref={inputRef} placeholder="Search..." />
      <button onClick={handleFocus}>Focus Input</button>
      <button onClick={handleSelect}>Select All</button>
    </div>
  );
}
```

**Output**
```
// Click "Focus Input":
← cursor appears in the input field

// Click "Select All":
← all text in input is highlighted
```

**Explanation**
`inputRef.current` holds the actual `<input>` DOM element. We call native DOM methods on it directly. No state is involved — the component does NOT re-render when focus or selection changes.

---

### 2. Auto-Focus on Mount

**Theory**
Combine `useRef` with `useEffect` to perform DOM operations after the component mounts — a very common pattern for modal dialogs, search bars, and login forms.

**Working Flow**

![flow-chart-2](flow-chart-2.png)

**Example**
```jsx
import { useRef, useEffect } from 'react';

function LoginModal({ isOpen }) {
  const emailRef = useRef(null);

  // Auto-focus the email input when modal opens
  useEffect(() => {
    if (isOpen && emailRef.current) {
      emailRef.current.focus();
    }
  }, [isOpen]);

  if (!isOpen) return null;

  return (
    <div className="modal">
      <input ref={emailRef} type="email" placeholder="Email" />
      <input type="password" placeholder="Password" />
      <button>Login</button>
    </div>
  );
}
```

**Output**
```
// isOpen becomes true:
← email input is automatically focused, cursor ready to type
```

---

### 3. Storing Previous Value

**Theory**
A ref persists across renders without causing them. This makes it perfect for remembering the value from the previous render — something impossible with regular variables (they reset) or state (would cause an extra render loop).

**Working Flow**

![flow-chart-3](flow-chart-3.png)

**Example**
```jsx
import { useState, useEffect, useRef } from 'react';

function PreviousValue({ value }) {
  const prevRef = useRef(null);

  useEffect(() => {
    // After render, store this render's value for the NEXT render
    prevRef.current = value;
  });

  return (
    <div>
      <p>Current:  {value}</p>
      <p>Previous: {prevRef.current ?? 'none'}</p>
    </div>
  );
}

function App() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>+1</button>
      <PreviousValue value={count} />
    </div>
  );
}
```

**Output**
```
Initial         → Current: 0 | Previous: none
After +1 click  → Current: 1 | Previous: 0
After +1 click  → Current: 2 | Previous: 1
After +1 click  → Current: 3 | Previous: 2
```

---

### 4. Storing Timer IDs — No Re-render Needed

**Theory**
Timer IDs from `setInterval` or `setTimeout` need to be stored somewhere to cancel them later. Putting them in state would cause an unnecessary re-render every time the timer starts/stops. A ref stores the ID silently.

**Working Flow**

![flow-chart-4](flow-chart-4.png)

**Example**
```jsx
import { useState, useRef } from 'react';

function Stopwatch() {
  const [time, setTime]      = useState(0);
  const [running, setRunning] = useState(false);
  const intervalRef           = useRef(null);

  function start() {
    if (running) return;
    setRunning(true);
    intervalRef.current = setInterval(() => {
      setTime(t => t + 1);
    }, 1000);
  }

  function stop() {
    clearInterval(intervalRef.current);
    setRunning(false);
  }

  function reset() {
    clearInterval(intervalRef.current);
    setRunning(false);
    setTime(0);
  }

  return (
    <div>
      <p>{time}s</p>
      <button onClick={start}>Start</button>
      <button onClick={stop}>Stop</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}
```

**Output**
```
// Click Start:
1s, 2s, 3s, ...   ← counts up every second

// Click Stop:
← timer pauses, no re-render caused by storing the interval ID

// Click Reset:
0s
```

---

### 5. useRef vs useState — When to Use Which

**Theory**
Both persist data across renders. The critical difference: `setState` triggers a re-render; changing `ref.current` does not.

**Working Flow**

![flow-chart-5](flow-chart-5.png)

**Example**
```jsx
import { useState, useRef } from 'react';

function Comparison() {
  const [stateCount, setStateCount] = useState(0);
  const refCount                    = useRef(0);

  function incrementState() {
    setStateCount(c => c + 1); // causes re-render → UI updates
  }

  function incrementRef() {
    refCount.current += 1;     // silent — NO re-render
    console.log('Ref value:', refCount.current);
  }

  return (
    <div>
      <p>State (visible in UI): {stateCount}</p>
      <p>Ref (only in console): — check console</p>
      <button onClick={incrementState}>Increment State</button>
      <button onClick={incrementRef}>Increment Ref</button>
    </div>
  );
}
```

**Output**
```
// Click "Increment State":
UI updates → "State (visible in UI): 1"

// Click "Increment Ref" 5 times:
Console: Ref value: 1
Console: Ref value: 2
Console: Ref value: 3
UI does NOT change → "Ref (only in console): —"
```

| | `useState` | `useRef` |
| :--- | :--- | :--- |
| Triggers re-render | ✅ Yes | ❌ No |
| Visible in JSX | ✅ Yes | ❌ Only if read manually |
| Mutable | Via setter | Directly (.current) |
| Use for | UI data | DOM refs, timers, prev values |

---

### 6. forwardRef — Passing Ref from Parent to Child

**Theory**
By default, you cannot attach a `ref` to a custom component. `React.forwardRef` lets a parent pass its ref down to a specific DOM element inside the child component.

**Working Flow**

![flow-chart-6](flow-chart-6.png)

**Example**
```jsx
import { useRef, forwardRef } from 'react';

// Child — wraps its input and forwards the ref to it
const FancyInput = forwardRef(function FancyInput(props, ref) {
  return (
    <input
      ref={ref}
      style={{ border: '2px solid blue', borderRadius: '4px' }}
      {...props}
    />
  );
});

// Parent — controls the child's DOM input directly
function Form() {
  const inputRef = useRef(null);

  return (
    <div>
      <FancyInput ref={inputRef} placeholder="Type here..." />
      <button onClick={() => inputRef.current.focus()}>
        Focus Input
      </button>
    </div>
  );
}
```

**Output**
```
// Click "Focus Input":
← cursor appears inside FancyInput's styled input element
```

---

[View Interview Questions](./interview.md)
