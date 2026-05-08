- Category: React Hooks
- Difficulty: Beginner
- Related: useReducer, useRef, useEffect

### useState — component memory that drives the UI
`useState` is React's way of giving a functional component a memory. Every piece of data that can change over time — a counter, a form field value, a loading flag — should live in state. When state changes, React automatically re-renders the component to reflect the new data.

**Analogy**
A whiteboard on a classroom wall. The teacher (React) looks at it before every class to decide what to draw on the screen. When you erase a number and write a new one (setState), the teacher notices and redraws the whole board to match. A regular JavaScript variable is a sticky note in your pocket — changing it doesn't update the board.

---

### 1. What useState is — state = component memory

**Theory**
`useState` returns a pair: the current value and a setter function. Calling the setter tells React "this value has changed — please re-render". Regular variables reset on every render; state persists across renders because React stores it outside the component function.

Syntax: `const [value, setValue] = useState(initialValue)`

**Working Flow**

![flow-chart](flow-chart.png)

**Example**
```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0); // initial value = 0

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
      <button onClick={() => setCount(count - 1)}>-1</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}
```

**Output**
```
Initial render  → Count: 0
After +1 click  → Count: 1
After +1 click  → Count: 2
After Reset     → Count: 0
```

**Explanation**
`useState(0)` stores the number `0` in React's memory. Each click calls `setCount` with a new value. React re-runs the component function with the updated `count`, producing a fresh render. The variable `count` inside the function always holds the value from the latest render.

---

### 2. Primitive state — number, string, boolean

**Theory**
State can hold any primitive: a number, a string, or a boolean. Each piece of state should be a single concern — don't mix unrelated values into one `useState` call unless they always change together.

**Working Flow**

![flow-chart-2](flow-chart-2.png)

**Example**
```jsx
import { useState } from 'react';

function PrimitiveExamples() {
  const [score, setScore]       = useState(0);
  const [name, setName]         = useState('');
  const [isVisible, setVisible] = useState(true);

  return (
    <div>
      {/* Number */}
      <p>Score: {score}</p>
      <button onClick={() => setScore(score + 10)}>+10 Points</button>

      {/* String */}
      <input
        value={name}
        onChange={e => setName(e.target.value)}
        placeholder="Enter name"
      />
      <p>Hello, {name || 'stranger'}!</p>

      {/* Boolean toggle */}
      <button onClick={() => setVisible(v => !v)}>
        {isVisible ? 'Hide' : 'Show'} Panel
      </button>
      {isVisible && <div className="panel">I am visible!</div>}
    </div>
  );
}
```

**Output**
```
Initial         → Score: 0 | Hello, stranger! | Panel visible
+10 Points      → Score: 10
type "Alice"    → Hello, Alice!
Hide Panel      → Panel hidden
Show Panel      → Panel visible
```

**Explanation**
Three independent pieces of state, each with one job. String state is driven by the `onChange` event of a controlled input — `value={name}` keeps the input in sync with state. Boolean state is toggled with a functional update `v => !v` so the toggle always flips the latest value.

---

### 3. Object state — spread pattern to avoid losing fields

**Theory**
When state is an object, the setter **replaces** the whole object — it does not merge fields automatically. Forgetting to spread the old state will cause other fields to disappear. Always spread first, then override only the changed key.

**Working Flow**

![flow-chart-3](flow-chart-3.png)

**Example**
```jsx
import { useState } from 'react';

function ProfileForm() {
  const [user, setUser] = useState({
    firstName: '',
    lastName: '',
    email: '',
  });

  // CORRECT — spread existing fields, override only the changed one
  const handleChange = (field, value) => {
    setUser(prev => ({ ...prev, [field]: value }));
  };

  // WRONG — this wipes firstName and lastName!
  // const handleChange = (field, value) => setUser({ [field]: value });

  return (
    <form>
      <input
        value={user.firstName}
        onChange={e => handleChange('firstName', e.target.value)}
        placeholder="First name"
      />
      <input
        value={user.lastName}
        onChange={e => handleChange('lastName', e.target.value)}
        placeholder="Last name"
      />
      <input
        value={user.email}
        onChange={e => handleChange('email', e.target.value)}
        placeholder="Email"
      />
      <p>Preview: {user.firstName} {user.lastName} — {user.email}</p>
    </form>
  );
}
```

**Output**
```
type "Jane" in First  → user = { firstName:"Jane", lastName:"", email:"" }
type "Doe"  in Last   → user = { firstName:"Jane", lastName:"Doe", email:"" }
type email            → user = { firstName:"Jane", lastName:"Doe", email:"jane@..." }
Preview               → Jane Doe — jane@example.com
```

**Explanation**
`[field]: value` is computed property syntax — it creates a key from the variable. The spread `...prev` copies all existing fields before we override just one. Without spread, each keystroke would erase the other fields.

---

### 4. Array state — add / remove / update without mutation

**Theory**
State arrays must never be mutated directly (`push`, `splice`, etc.) because React compares old and new state by reference. If you mutate the same array, React sees the same reference and skips the re-render. Always create a new array with spread, `filter`, `map`, or `concat`.

**Working Flow**

![flow-chart-4](flow-chart-4.png)

**Example**
```jsx
import { useState } from 'react';

function TodoList() {
  const [todos, setTodos] = useState([
    { id: 1, text: 'Buy groceries', done: false },
    { id: 2, text: 'Write code',    done: false },
  ]);
  const [input, setInput] = useState('');

  // ADD — spread old array, append new item
  const addTodo = () => {
    if (!input.trim()) return;
    setTodos(prev => [
      ...prev,
      { id: Date.now(), text: input, done: false },
    ]);
    setInput('');
  };

  // REMOVE — filter returns a new array without the item
  const removeTodo = id => {
    setTodos(prev => prev.filter(t => t.id !== id));
  };

  // UPDATE — map returns a new array with one item changed
  const toggleDone = id => {
    setTodos(prev =>
      prev.map(t => t.id === id ? { ...t, done: !t.done } : t)
    );
  };

  return (
    <div>
      <input value={input} onChange={e => setInput(e.target.value)} />
      <button onClick={addTodo}>Add</button>
      <ul>
        {todos.map(t => (
          <li key={t.id}>
            <span style={{ textDecoration: t.done ? 'line-through' : 'none' }}>
              {t.text}
            </span>
            <button onClick={() => toggleDone(t.id)}>✓</button>
            <button onClick={() => removeTodo(t.id)}>✕</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

**Output**
```
Initial         → ["Buy groceries", "Write code"]
Add "Sleep"     → ["Buy groceries", "Write code", "Sleep"]
Toggle id=1     → "Buy groceries" gets line-through
Remove id=2     → ["Buy groceries", "Sleep"]
```

**Explanation**
Three common operations, three different non-mutating techniques: spread+append for add, `filter` for remove, `map`+spread for update. In each case a brand new array is returned, so React detects the change and re-renders.

---

### 5. Functional update form — `prev => prev + 1` and why it matters

**Theory**
React may batch multiple state updates together. If two updates run before a re-render, the second one that reads `count` will see the stale value from the current render, not the result of the first update. The functional form `setCount(prev => prev + 1)` receives the most recent queued value instead of the captured render-time snapshot — it is always safe when the new state depends on the old.

**Working Flow**

![flow-chart-5](flow-chart-5.png)

**Example**
```jsx
import { useState } from 'react';

function BatchDemo() {
  const [count, setCount] = useState(0);

  // WRONG — both reads use the same stale `count` snapshot
  const incrementTwiceWrong = () => {
    setCount(count + 1); // reads count = 0 → schedules 1
    setCount(count + 1); // reads count = 0 → schedules 1 again!
    // final result: 1, not 2
  };

  // CORRECT — each receives the latest queued value
  const incrementTwiceRight = () => {
    setCount(prev => prev + 1); // queued: 0 → 1
    setCount(prev => prev + 1); // queued: 1 → 2
    // final result: 2
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={incrementTwiceWrong}>+2 (Wrong)</button>
      <button onClick={incrementTwiceRight}>+2 (Correct)</button>
    </div>
  );
}
```

**Output**
```
Start: Count = 0
Click "+2 (Wrong)"   → Count = 1  ← bug!
Click "+2 (Correct)" → Count = 3  (1 + 2)
```

**Explanation**
The stale closure problem: when you write `setCount(count + 1)` twice in one handler, both calls capture the same `count` value from the current render. The functional form `prev => prev + 1` tells React "apply this transformation to whatever the latest value is", making sequential updates safe even inside batched renders or async code.

---

### 6. Lazy initializer — expensive initial value computed once

**Theory**
The second argument to `useState` is the **initial value** and is normally only used on the first render. However, if you write `useState(expensiveFunction())`, JavaScript calls `expensiveFunction` on every render — the result is just discarded after mount. Pass the **function itself** (not its result) to avoid this: `useState(expensiveFunction)`. React calls it once during mount.

**Working Flow**

![flow-chart-6](flow-chart-6.png)

**Example**
```jsx
import { useState } from 'react';

// Simulates reading from localStorage — should only run once
function loadSavedTheme() {
  console.log('Reading localStorage...');
  return localStorage.getItem('theme') || 'light';
}

function ThemeApp() {
  // WRONG — loadSavedTheme() called on EVERY render
  // const [theme, setTheme] = useState(loadSavedTheme());

  // CORRECT — loadSavedTheme passed as a reference, called ONCE
  const [theme, setTheme] = useState(loadSavedTheme);

  const toggle = () => {
    const next = theme === 'light' ? 'dark' : 'light';
    setTheme(next);
    localStorage.setItem('theme', next);
  };

  return (
    <div className={`app theme-${theme}`}>
      <p>Current theme: {theme}</p>
      <button onClick={toggle}>Toggle Theme</button>
    </div>
  );
}
```

**Output**
```
Mount           → "Reading localStorage..." (logged ONCE)
Toggle          → theme switches, localStorage updated, no extra log
Toggle again    → theme switches again, still no extra log
```

**Explanation**
`useState(loadSavedTheme)` — no parentheses — passes the function as a reference. React stores the function and calls it exactly once during the initial render. Every subsequent re-render ignores it. This is critical when the initializer does something costly like reading `localStorage`, parsing JSON, or iterating large datasets.

---

### Real-world: Multi-field form with validation state

A registration form that combines object state for fields, boolean state for submission, and array state for error messages — all together in one real component.

```jsx
import { useState } from 'react';

const EMPTY_FORM = { username: '', email: '', password: '' };

function RegistrationForm() {
  const [fields, setFields]     = useState(EMPTY_FORM);
  const [errors, setErrors]     = useState([]);
  const [submitted, setSubmitted] = useState(false);

  const handleChange = e => {
    const { name, value } = e.target;
    setFields(prev => ({ ...prev, [name]: value }));
  };

  const validate = () => {
    const errs = [];
    if (!fields.username.trim()) errs.push('Username is required.');
    if (!fields.email.includes('@')) errs.push('Valid email required.');
    if (fields.password.length < 6) errs.push('Password must be ≥ 6 chars.');
    return errs;
  };

  const handleSubmit = e => {
    e.preventDefault();
    const errs = validate();
    setErrors(errs);
    if (errs.length === 0) {
      setSubmitted(true);
    }
  };

  if (submitted) return <p>Welcome, {fields.username}!</p>;

  return (
    <form onSubmit={handleSubmit}>
      {errors.map((err, i) => (
        <p key={i} style={{ color: 'red' }}>{err}</p>
      ))}
      <input name="username" value={fields.username} onChange={handleChange} placeholder="Username" />
      <input name="email"    value={fields.email}    onChange={handleChange} placeholder="Email" />
      <input name="password" value={fields.password} onChange={handleChange} placeholder="Password" type="password" />
      <button type="submit">Register</button>
    </form>
  );
}
```

**Output**
```
Submit empty    → ["Username is required.", "Valid email required.", "Password must be ≥ 6 chars."]
Fill correctly  → Welcome, alice!
```

This pattern is used in virtually every real-world React application. The key insight: each concern has its own piece of state — fields (object), errors (array), submitted (boolean) — making the logic predictable and easy to extend.

---

[View Interview Questions](./interview.md)
