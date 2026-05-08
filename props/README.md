- Category: React Core
- Difficulty: Beginner
- Related: state, props-vs-state, props-drilling, context-api

### React Props — passing data into components

Props (short for *properties*) are the mechanism React uses to pass data **down** from a parent component to a child component. A component receives props as a plain JavaScript object and uses those values to decide what to render.

**Analogy**
A vending machine: props are the coins and button selection you feed in. The machine (component) uses those inputs and produces a drink (rendered UI). You cannot reach inside and rewire the machine from outside — that is what makes props *read-only*.

---

### 1. Passing and Receiving Props

**Theory**
Props are written as HTML-like attributes on the JSX element in the parent. The child receives them as a single object — conventionally called `props`. Every attribute you write becomes a key on that object.

**Working Flow**
![flow-chart](flow-chart.png)

**Example**
```jsx
// Parent
function App() {
  return <Greeting name="Alice" age={28} />;
}

// Child
function Greeting(props) {
  return (
    <h1>
      Hello, {props.name}. You are {props.age} years old.
    </h1>
  );
}
```

**Output**
```
Rendered UI:
  Hello, Alice. You are 28 years old.
```

**Explanation**
- String props use quotes: `name="Alice"`
- Non-string values use curly braces: `age={28}`, `active={true}`, `style={{ color: "red" }}`
- Inside the child, `props` is just a plain JS object — read it like any object.
- Props are **read-only**: the child must never do `props.name = "Bob"`.

---

### 2. Destructuring Props

**Theory**
Instead of writing `props.name` repeatedly, you can destructure the props object right in the function signature. This keeps JSX cleaner and reduces repetition.

**Working Flow**
![flow-chart-2](flow-chart-2.png)

**Example**
```jsx
// Without destructuring (verbose)
function Card(props) {
  return <div>{props.title} — ${props.price}</div>;
}

// With destructuring (clean)
function Card({ title, price, inStock }) {
  return (
    <div>
      <h2>{title}</h2>
      <p>${price}</p>
      <span>{inStock ? "In Stock" : "Sold Out"}</span>
    </div>
  );
}

// Usage
<Card title="Wireless Mouse" price={29} inStock={true} />
```

**Output**
```
Rendered UI:
  Wireless Mouse
  $29
  In Stock
```

**Explanation**
Destructuring is plain ES6 JavaScript — it is identical to writing `const { title, price } = props` at the top of the body. You can also rename on the way out: `{ title: productName }`.

---

### 3. Default Props

**Theory**
Provide fallback values for props that might not be passed by the parent. Use ES6 default parameter syntax directly in the function signature. Defaults only apply when the prop is `undefined` — not when it is `null` or `false`.

**Working Flow**
![flow-chart-3](flow-chart-3.png)

**Example**
```jsx
function Button({ label = "Click Me", color = "blue", disabled = false }) {
  return (
    <button
      style={{ backgroundColor: color, opacity: disabled ? 0.5 : 1 }}
      disabled={disabled}
    >
      {label}
    </button>
  );
}

<Button />                                 // all defaults
<Button label="Submit" color="green" />    // overrides two
<Button label="Loading..." disabled={true} /> // overrides disabled
```

**Output**
```
<Button />                             → blue button labelled "Click Me"
<Button label="Submit" color="green">  → green button labelled "Submit"
<Button label="Loading..." disabled>   → faded blue button, not clickable
```

**Explanation**
Default props prevent `undefined` from appearing in your UI without any extra `if` checks. They serve as documentation too — they communicate what is optional.

---

### 4. Passing Objects, Arrays, and Functions as Props

**Theory**
Any JavaScript value can be a prop — primitives, objects, arrays, and functions. Passing a function lets the child trigger logic that lives in the parent. This is called the *callback* pattern.

**Working Flow**
![flow-chart-4](flow-chart-4.png)

**Example**
```jsx
// Passing an object
function UserCard({ user }) {
  return <p>{user.name} — {user.email}</p>;
}
<UserCard user={{ name: "Bob", email: "bob@example.com" }} />

// Passing an array
function TagList({ tags }) {
  return (
    <ul>
      {tags.map(tag => <li key={tag}>{tag}</li>)}
    </ul>
  );
}
<TagList tags={["React", "JavaScript", "CSS"]} />

// Passing a function (callback)
function App() {
  function handleDelete() {
    console.log("Item deleted!");
  }
  return <DeleteButton onDelete={handleDelete} />;
}

function DeleteButton({ onDelete }) {
  return <button onClick={onDelete}>Delete</button>;
}
```

**Output**
```
UserCard renders:    Bob — bob@example.com

TagList renders:
  • React
  • JavaScript
  • CSS

DeleteButton click → console: "Item deleted!"
```

**Explanation**
Functions passed as props let child components trigger actions in the parent while the child itself owns no logic. This is how React maintains unidirectional data flow — data goes *down*, events go *up*.

---

### 5. The `children` Prop

**Theory**
Whatever JSX you place *between* a component's opening and closing tags becomes `props.children`. This lets you build wrapper/layout components that are agnostic about their inner content.

**Working Flow**
![flow-chart-5](flow-chart-5.png)

**Example**
```jsx
function Card({ title, children }) {
  return (
    <div style={{ border: "1px solid #ccc", padding: "16px", borderRadius: "8px" }}>
      <h2 style={{ marginTop: 0 }}>{title}</h2>
      <div>{children}</div>
    </div>
  );
}

function App() {
  return (
    <Card title="User Profile">
      <p>Name: Alice</p>
      <p>Role: Admin</p>
      <button>Edit Profile</button>
    </Card>
  );
}
```

**Output**
```
Rendered UI:
┌─────────────────────────┐
│ User Profile            │
│  Name: Alice            │
│  Role: Admin            │
│  [Edit Profile]         │
└─────────────────────────┘
```

**Explanation**
`children` can be a single element, multiple elements, a string, a number, or `null`. It is ideal for modal dialogs, layout panels, and reusable containers. Anything placed between the tags lands in `children`.

---

### 6. Callback Props — Lifting State Up

**Theory**
When a child component needs to update data owned by its parent, the parent passes a function down as a prop. The child calls that function. Data flows one way: downward via props, upward via callbacks.

**Working Flow**
![flow-chart-6](flow-chart-6.png)

**Example**
```jsx
function App() {
  const [count, setCount] = React.useState(0);

  function handleIncrement() {
    setCount(prev => prev + 1);
  }

  return (
    <div>
      <p>Total Clicks: {count}</p>
      <CounterButton onIncrement={handleIncrement} />
    </div>
  );
}

// Child owns no state — just calls the function it received
function CounterButton({ onIncrement }) {
  return <button onClick={onIncrement}>+1</button>;
}
```

**Output**
```
Initial render:  Total Clicks: 0   [+1]
After 1 click:   Total Clicks: 1   [+1]
After 3 clicks:  Total Clicks: 3   [+1]
```

**Explanation**
The child is *controlled* by the parent. It has no idea what happens when the button is clicked — it simply calls the function it received. This pattern is the foundation of React's unidirectional data flow and "lifting state up".

---

### 7. PropTypes Validation

**Theory**
`PropTypes` is a runtime type-checking library (via the `prop-types` package). It warns in the browser console when a component receives the wrong type or is missing a required prop. It only runs in development, not production.

**Working Flow**
![flow-chart-7](flow-chart-7.png)

**Example**
```jsx
import PropTypes from "prop-types";

function UserCard({ name, age, onSelect, isAdmin }) {
  return (
    <div onClick={onSelect}>
      {name} (age {age}) {isAdmin && " — Admin"}
    </div>
  );
}

UserCard.propTypes = {
  name:     PropTypes.string.isRequired,
  age:      PropTypes.number.isRequired,
  onSelect: PropTypes.func,
  isAdmin:  PropTypes.bool,
};

UserCard.defaultProps = {
  isAdmin: false,
};

// Correct usage — no warnings
<UserCard name="Alice" age={28} onSelect={() => {}} />

// Wrong type — age is a string
<UserCard name="Bob" age="30" />

// Missing required prop
<UserCard age={25} />
```

**Output**
```
Console (wrong type):
  Warning: Failed prop type: Invalid prop `age` of type `string`
  supplied to `UserCard`, expected `number`.

Console (missing required):
  Warning: Failed prop type: The prop `name` is marked as required
  in `UserCard`, but its value is `undefined`.
```

---

### 8. Prop Drilling (Intro)

**Theory**
Prop drilling happens when data must pass through several intermediate components that do not actually use it — they only forward it to the next level. This makes code brittle: adding or removing a prop requires editing every layer.

**Working Flow**
![flow-chart-8](flow-chart-8.png)

**Example**
```jsx
function App() {
  const user = { name: "Alice", avatar: "alice.png" };
  return <Layout user={user} />;
}

function Layout({ user }) {        // does not use user, just forwards
  return <Sidebar user={user} />;
}

function Sidebar({ user }) {       // does not use user, just forwards
  return <UserAvatar user={user} />;
}

function UserAvatar({ user }) {    // finally uses it
  return <img src={user.avatar} alt={user.name} />;
}
```

**Output**
```
Rendered: <img src="alice.png" alt="Alice" />

Problem:  Layout and Sidebar are coupled to the user prop
          even though they never use it.
Solution: Context API — broadcast data directly to any consumer
          without threading it through intermediate layers.
```

---

### Real-World Example — Reusable Button Component

```jsx
import PropTypes from "prop-types";

function Button({ label, variant, size, onClick, disabled }) {
  const base = { borderRadius: "4px", fontWeight: "600", cursor: "pointer", border: "none" };

  const variantStyle = {
    primary:   { background: "#2563EB", color: "#fff" },
    secondary: { background: "#E5E7EB", color: "#374151" },
    danger:    { background: "#DC2626", color: "#fff" },
  }[variant];

  const sizeStyle = {
    sm: { padding: "6px 12px", fontSize: "14px" },
    md: { padding: "10px 20px", fontSize: "16px" },
    lg: { padding: "14px 28px", fontSize: "18px" },
  }[size];

  return (
    <button
      style={{ ...base, ...variantStyle, ...sizeStyle, opacity: disabled ? 0.5 : 1 }}
      onClick={onClick}
      disabled={disabled}
    >
      {label}
    </button>
  );
}

Button.propTypes = {
  label:    PropTypes.string.isRequired,
  variant:  PropTypes.oneOf(["primary", "secondary", "danger"]),
  size:     PropTypes.oneOf(["sm", "md", "lg"]),
  onClick:  PropTypes.func,
  disabled: PropTypes.bool,
};

Button.defaultProps = {
  variant:  "primary",
  size:     "md",
  disabled: false,
};

// Usage across the app
<Button label="Save Changes" onClick={handleSave} />
<Button label="Cancel" variant="secondary" size="sm" onClick={handleCancel} />
<Button label="Delete Account" variant="danger" size="lg" onClick={handleDelete} />
<Button label="Loading..." disabled={true} />
```

**Output**
```
[Save Changes]       → blue medium button
[Cancel]             → gray small button
[Delete Account]     → red large button
[Loading...]         → faded blue medium button, not clickable
```

---

[View Interview Questions](./interview.md)
