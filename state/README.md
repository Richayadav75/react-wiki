- Category: React Core
- Difficulty: Beginner
- Related: props, props-vs-state, hooks, useState, useReducer

### React State — a component's private, mutable memory

State is data that a component owns and can change over time. Unlike props (read-only, given by a parent), state is managed *inside* the component itself. When state changes, React re-renders the component so the UI stays in sync with the data.

**Analogy**
A whiteboard on your desk. You can write on it, erase it, and update it any time — but it belongs only to you. Props are like a sticky note handed to you by someone else: you can read it, but you cannot change what they wrote.

---

### 1. useState Basics

**Theory**
`useState` is the hook that adds state to a functional component. It returns exactly two things: the current value and a setter function. Calling the setter triggers React to re-render the component with the new value.

**Working Flow**
![flow-chart](flow-chart.png)

**Example**
```jsx
import { useState } from "react";

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
Initial render:   Count: 0   [+1] [-1] [Reset]
After +1 click:   Count: 1
After +1 click:   Count: 2
After Reset:      Count: 0
```

**Explanation**
- `useState(0)` — the argument is the *initial* value, used only on the first render.
- `setCount(count + 1)` — calling the setter schedules a re-render. React will call your component again with the new value.
- Never do `count++` or `count = 5` — React will not detect the change and the UI stays stale.

---

### 2. Never Mutate State Directly

**Theory**
React detects state changes by reference comparison. If you mutate a value in place (same reference), React sees no change and skips the re-render. Always produce a *new* value or a new object/array.

**Working Flow**
![flow-chart-2](flow-chart-2.png)

**Example**
```jsx
// WRONG — mutating directly
const [items, setItems] = useState(["a", "b"]);
items.push("c");        // mutates array in place
setItems(items);        // React sees the same reference — may skip re-render

// CORRECT — produce a new array
setItems([...items, "c"]);   // new array reference → guaranteed re-render

// WRONG — mutating object
const [user, setUser] = useState({ name: "Alice", age: 28 });
user.name = "Bob";      // mutates object in place
setUser(user);          // same reference — may skip re-render

// CORRECT — spread into new object
setUser({ ...user, name: "Bob" });  // new object → guaranteed re-render
```

**Output**
```
WRONG  approach: UI may not update even though data changed
CORRECT approach: UI always reflects the latest state
```

---

### 3. Functional Updates — prev => prev + 1

**Theory**
When your new state depends on the old state, always pass a *function* to the setter instead of a computed value. This guarantees you read the most up-to-date state, even when multiple updates are batched together (the stale closure problem).

**Working Flow**
![flow-chart-3](flow-chart-3.png)

**Example**
```jsx
import { useState } from "react";

function FastCounter() {
  const [count, setCount] = useState(0);

  function addThree() {
    // Functional updates — each reads the result of the previous
    setCount(prev => prev + 1);
    setCount(prev => prev + 1);
    setCount(prev => prev + 1);
  }

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={addThree}>+3 at once</button>
    </div>
  );
}
```

**Output**
```
Count: 0
[Click "+3 at once"]
Count: 3   ← correct, not Count: 1
```

**Explanation**
Use `prev => prev + 1` any time the new value depends on the old value. For simple cases where a button is clicked once per event, either form works — but the functional form is always safer.

---

### 4. Object State with Spread

**Theory**
State can hold a plain object (e.g., a form with multiple fields). Since `useState` replaces the entire state value (unlike class `this.setState` which merges), you must spread the existing object and only override the field that changed.

**Working Flow**
![flow-chart-4](flow-chart-4.png)

**Example**
```jsx
import { useState } from "react";

function UserForm() {
  const [user, setUser] = useState({ name: "", email: "", age: "" });

  function handleChange(e) {
    const { name, value } = e.target;
    setUser(prev => ({ ...prev, [name]: value })); // computed property key
  }

  return (
    <form>
      <input name="name"  value={user.name}  onChange={handleChange} placeholder="Name" />
      <input name="email" value={user.email} onChange={handleChange} placeholder="Email" />
      <input name="age"   value={user.age}   onChange={handleChange} placeholder="Age" />
      <p>Hello, {user.name || "stranger"}!</p>
    </form>
  );
}
```

**Output**
```
Type "Alice" in Name  → p shows "Hello, Alice!"
Type "alice@dev.com" in Email → name preserved, email updated
Type "28" in Age      → name and email preserved, age updated
```

---

### 5. Array State — Add, Remove, Update

**Theory**
Arrays in state must be treated immutably. Never call `push`, `pop`, `splice`, or `sort` on the state array. Instead, produce a new array using spread, `filter`, `map`, or `concat`.

**Working Flow**
![flow-chart-5](flow-chart-5.png)

**Example**
```jsx
import { useState } from "react";

function TodoList() {
  const [todos, setTodos] = useState([
    { id: 1, text: "Learn React", done: false },
  ]);
  const [input, setInput] = useState("");
  let nextId = todos.length + 1;

  function addTodo() {
    if (!input.trim()) return;
    setTodos(prev => [...prev, { id: nextId++, text: input, done: false }]);
    setInput("");
  }

  function toggleTodo(id) {
    setTodos(prev =>
      prev.map(t => t.id === id ? { ...t, done: !t.done } : t)
    );
  }

  function removeTodo(id) {
    setTodos(prev => prev.filter(t => t.id !== id));
  }

  return (
    <div>
      <input value={input} onChange={e => setInput(e.target.value)} placeholder="New task" />
      <button onClick={addTodo}>Add</button>
      <ul>
        {todos.map(t => (
          <li key={t.id} style={{ textDecoration: t.done ? "line-through" : "none" }}>
            <input type="checkbox" checked={t.done} onChange={() => toggleTodo(t.id)} />
            {t.text}
            <button onClick={() => removeTodo(t.id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

**Output**
```
Initial:   [ ] Learn React   [Delete]

Type "Learn Hooks" → click Add:
           [ ] Learn React   [Delete]
           [ ] Learn Hooks   [Delete]

Check "Learn React":
           [x] ~~Learn React~~ [Delete]
           [ ] Learn Hooks    [Delete]

Click Delete on "Learn React":
           [ ] Learn Hooks   [Delete]
```

---

### 6. Lifting State Up

**Theory**
When two sibling components need to share and synchronise the same data, move (lift) the state to their closest common parent. The parent owns the state and passes it down via props. Only one component manages the truth.

**Working Flow**
![flow-chart-6](flow-chart-6.png)

**Example**
```jsx
import { useState } from "react";

function App() {
  const [text, setText] = useState("");

  return (
    <div>
      <InputBox value={text} onChange={setText} />
      <Preview value={text} />
      <CharCount value={text} />
    </div>
  );
}

function InputBox({ value, onChange }) {
  return (
    <textarea
      value={value}
      onChange={e => onChange(e.target.value)}
      placeholder="Type something..."
    />
  );
}

function Preview({ value }) {
  return <div style={{ border: "1px solid #ccc" }}>{value || "Nothing yet"}</div>;
}

function CharCount({ value }) {
  return <p>{value.length} characters</p>;
}
```

**Output**
```
Type "Hello":
  [Textarea: "Hello"]
  Preview box: Hello
  5 characters

All three components stay in sync — one source of truth in App.
```

---

### 7. What Belongs in State and What Does Not

**Theory**
State should hold data that: (1) is not computable from existing state or props, (2) changes over time due to user interaction or async operations, and (3) directly affects what is rendered. Everything else should be derived or stored elsewhere.

**Working Flow**
![flow-chart-7](flow-chart-7.png)

**Example**
```jsx
// BAD — firstName and lastName in state AND fullName also in state
const [firstName, setFirstName] = useState("Alice");
const [lastName,  setLastName]  = useState("Smith");
const [fullName,  setFullName]  = useState("Alice Smith"); // redundant!

// GOOD — derive fullName instead of storing it
const [firstName, setFirstName] = useState("Alice");
const [lastName,  setLastName]  = useState("Smith");
const fullName = `${firstName} ${lastName}`;  // computed, no state needed

// BAD — putting a prop directly into state (gets out of sync)
function Child({ initialCount }) {
  const [count, setCount] = useState(initialCount); // only syncs on first render!
}

// GOOD — use the prop directly or use it only as initial value intentionally
function Child({ count }) {
  return <p>{count}</p>; // just read the prop
}
```

**Output**
```
GOOD fullName: always "Alice Smith" when both are "Alice" + "Smith"
BAD  fullName: could be "Alice Smith" in state while firstName changed to "Bob"
               → fullName still shows "Alice Smith" (stale!)
```

---

### Real-World Example — Form with Validation

```jsx
import { useState } from "react";

function SignupForm() {
  const [form, setForm] = useState({ email: "", password: "" });
  const [errors, setErrors] = useState({});
  const [submitted, setSubmitted] = useState(false);

  function handleChange(e) {
    const { name, value } = e.target;
    setForm(prev => ({ ...prev, [name]: value }));
  }

  function validate() {
    const errs = {};
    if (!form.email.includes("@")) errs.email = "Invalid email";
    if (form.password.length < 6)  errs.password = "Min 6 characters";
    return errs;
  }

  function handleSubmit(e) {
    e.preventDefault();
    const errs = validate();
    if (Object.keys(errs).length > 0) {
      setErrors(errs);
    } else {
      setSubmitted(true);
    }
  }

  if (submitted) return <p>Welcome, {form.email}!</p>;

  return (
    <form onSubmit={handleSubmit}>
      <input name="email"    value={form.email}    onChange={handleChange} placeholder="Email" />
      {errors.email    && <span style={{ color: "red" }}>{errors.email}</span>}

      <input name="password" value={form.password} onChange={handleChange} type="password" placeholder="Password" />
      {errors.password && <span style={{ color: "red" }}>{errors.password}</span>}

      <button type="submit">Sign Up</button>
    </form>
  );
}
```

**Output**
```
Submit with "abc" and "123":
  Invalid email
  Min 6 characters

Submit with "alice@dev.com" and "secure123":
  Welcome, alice@dev.com!
```

---

[View Interview Questions](./interview.md)
