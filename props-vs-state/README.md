- Category: React Core
- Difficulty: Beginner
- Related: props, state, hooks, context-api

### Props vs State — two types of data, two different jobs

Props and state are the two pillars of data in React. Both can influence what a component renders, and both can trigger a re-render when they change. But they have opposite ownership models and mutability rules.

**Analogy**
Props are like the script handed to an actor by the director — the actor reads and follows it, but cannot rewrite it. State is the actor's own private notebook — they can update it between scenes, and it changes how they perform.

---

### 1. Side-by-Side Comparison

**Theory**
The fundamental difference is *ownership*. Props belong to the parent; state belongs to the component. This single distinction drives every other difference.

**Working Flow**
![flow-chart](flow-chart.png)

**Comparison Table**

| Feature            | Props                              | State                              |
|--------------------|------------------------------------|------------------------------------|
| Owner              | Parent component                   | The component itself               |
| Mutability         | Read-only (immutable)              | Mutable (via setter function)      |
| Who can change it? | Only the parent                    | Only the component that owns it    |
| Direction          | Top-down (parent → child)          | Local, can be passed down as props |
| Triggers re-render | Yes, when parent re-renders        | Yes, when setter is called         |
| Purpose            | Configure / share data             | Track changing, interactive data   |
| Initialized by     | Caller (JSX attributes)            | useState initial value             |

---

### 2. Props in Action — configured from outside

**Theory**
Props make components reusable. The same component can render completely different output depending on what the parent passes in. The component itself is stateless — it just transforms props into UI.

**Working Flow**
![flow-chart-2](flow-chart-2.png)

**Example**
```jsx
function Button({ label, color, onClick }) {
  return (
    <button style={{ backgroundColor: color }} onClick={onClick}>
      {label}
    </button>
  );
}

function App() {
  return (
    <div>
      <Button label="Save"   color="green" onClick={() => console.log("saved")} />
      <Button label="Delete" color="red"   onClick={() => console.log("deleted")} />
      <Button label="Cancel" color="gray"  onClick={() => console.log("cancelled")} />
    </div>
  );
}
```

**Output**
```
[Save]    → green button  (click → "saved")
[Delete]  → red button    (click → "deleted")
[Cancel]  → gray button   (click → "cancelled")

Same Button component, three different looks and behaviours.
```

---

### 3. State in Action — managed from inside

**Theory**
State gives a component memory. A component with state can respond to user interaction and update its own output independently. No parent coordination needed.

**Working Flow**
![flow-chart-3](flow-chart-3.png)

**Example**
```jsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(prev => prev + 1)}>+1</button>
      <button onClick={() => setCount(prev => prev - 1)}>-1</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}
```

**Output**
```
Count: 0   [+1] [-1] [Reset]
Click +1 → Count: 1
Click +1 → Count: 2
Click -1 → Count: 1
Click Reset → Count: 0
```

---

### 4. The Common Mistake — Copying Props into State

**Theory**
A very common bug: the parent passes a prop, and the child puts that prop value into its own state as the initial value. This creates two copies of the same data that can get out of sync. If the parent's value changes later, the child's state remains stuck at the original value.

**Working Flow**
![flow-chart-4](flow-chart-4.png)

**Example**
```jsx
// WRONG — state mirrors prop, gets out of sync
function NameDisplay({ name }) {
  const [displayName, setDisplayName] = useState(name); // copies prop once
  // If parent changes `name` prop later, displayName stays stale
  return <p>{displayName}</p>;
}

// CORRECT option A — just read the prop directly
function NameDisplay({ name }) {
  return <p>{name}</p>; // always reflects the latest prop
}

// CORRECT option B — intentional "uncontrolled" initial value
function EditableField({ initialValue, onSave }) {
  const [value, setValue] = useState(initialValue); // child "takes ownership"
  return (
    <>
      <input value={value} onChange={e => setValue(e.target.value)} />
      <button onClick={() => onSave(value)}>Save</button>
    </>
  );
}
```

**Output**
```
WRONG: Parent changes name prop from "Alice" to "Bob"
       → NameDisplay still shows "Alice" (stale state)

CORRECT option A: Parent changes name prop → NameDisplay immediately shows "Bob"
```

---

### 5. Controlled vs Uncontrolled Components

**Theory**
A *controlled* component has its form input value fully managed by React state. The displayed value comes from state; every keystroke triggers a state update. An *uncontrolled* component lets the DOM manage the value (accessed via `ref`). Controlled is the React-idiomatic approach.

**Working Flow**
![flow-chart-5](flow-chart-5.png)

**Example**
```jsx
// CONTROLLED — React owns the value
function ControlledInput() {
  const [value, setValue] = useState("");

  return (
    <>
      <input
        value={value}                          // value comes from state
        onChange={e => setValue(e.target.value)} // every keystroke updates state
      />
      <p>You typed: {value}</p>
    </>
  );
}

// UNCONTROLLED — DOM owns the value
import { useRef } from "react";

function UncontrolledInput() {
  const inputRef = useRef(null);

  function handleSubmit() {
    console.log("Value:", inputRef.current.value); // read on demand
  }

  return (
    <>
      <input ref={inputRef} defaultValue="" />
      <button onClick={handleSubmit}>Submit</button>
    </>
  );
}
```

**Output**
```
Controlled: every keystroke → state updates → "You typed: H", "You typed: Hi"
Uncontrolled: nothing shown until Submit clicked → console: "Value: Hi"
```

---

### 6. Single Source of Truth

**Theory**
When the same data is displayed in multiple places, it should be stored in exactly one place — typically the closest common parent. All other components read from that single source. This principle prevents inconsistency.

**Working Flow**
![flow-chart-6](flow-chart-6.png)

**Example**
```jsx
// SINGLE SOURCE OF TRUTH — App owns the form value
function App() {
  const [email, setEmail] = useState("");

  return (
    <div>
      <EmailInput  value={email} onChange={setEmail} />
      <EmailPreview email={email} />
      <SubmitButton email={email} />
    </div>
  );
}

function EmailInput({ value, onChange }) {
  return (
    <input
      type="email"
      value={value}
      onChange={e => onChange(e.target.value)}
      placeholder="Enter email"
    />
  );
}

function EmailPreview({ email }) {
  return <p>Will send to: {email || "nobody yet"}</p>;
}

function SubmitButton({ email }) {
  return (
    <button disabled={!email.includes("@")}>
      Send
    </button>
  );
}
```

**Output**
```
Type "alice@dev.com":
  Input shows:    alice@dev.com
  Preview shows:  Will send to: alice@dev.com
  Button:         [Send] (enabled, @ is present)

All three components read from the same state in App.
No duplication, no risk of inconsistency.
```

---

### 7. When to Use Props and When to Use State

**Theory**
Apply a simple checklist to each piece of data in your component to decide where it belongs.

**Working Flow**
![flow-chart-7](flow-chart-7.png)

**Example**
```
// PROPS — given by parent, static from child's perspective
<ProductCard
  name="Laptop"
  price={999}
  onAddToCart={handleAdd}
/>

// STATE — changes based on interaction
const [isWishlisted, setIsWishlisted] = useState(false);
const [quantity,     setQuantity]     = useState(1);

// DERIVED — computable, no need to store
const totalPrice = price * quantity;   // not in state, just computed
```

---

### Real-World Example — Form Controlled by Parent

```jsx
import { useState } from "react";

// Parent controls all values via state
function LoginPage() {
  const [form, setForm] = useState({ email: "", password: "" });
  const [isLoading, setIsLoading] = useState(false);

  function handleChange(field, value) {
    setForm(prev => ({ ...prev, [field]: value }));
  }

  async function handleSubmit() {
    setIsLoading(true);
    await fakeLogin(form); // simulate API call
    setIsLoading(false);
  }

  return (
    <LoginForm
      email={form.email}
      password={form.password}
      isLoading={isLoading}
      onEmailChange={val => handleChange("email", val)}
      onPasswordChange={val => handleChange("password", val)}
      onSubmit={handleSubmit}
    />
  );
}

// Child is a "dumb" form — receives everything via props
function LoginForm({ email, password, isLoading, onEmailChange, onPasswordChange, onSubmit }) {
  return (
    <form onSubmit={e => { e.preventDefault(); onSubmit(); }}>
      <input value={email}    onChange={e => onEmailChange(e.target.value)}    placeholder="Email" />
      <input value={password} onChange={e => onPasswordChange(e.target.value)} type="password" placeholder="Password" />
      <button type="submit" disabled={isLoading}>
        {isLoading ? "Logging in..." : "Login"}
      </button>
    </form>
  );
}
```

**Output**
```
User types email → onEmailChange fires → LoginPage state updates
  → email prop flows back down to LoginForm → input shows latest value

Click Login:
  Button → "Logging in..."   (isLoading = true)
  After API → "Login"        (isLoading = false)

Parent owns all data; child only displays and signals changes.
```

---

[View Interview Questions](./interview.md)
