# Props Interview Questions

---

**1. What are props in React and why are they read-only?**

Props are the mechanism for passing data from a parent component to a child component. They are read-only because React enforces unidirectional data flow — only the owner of the data should modify it. If children could mutate props, updates would become unpredictable and hard to trace.

```jsx
function Child({ name }) {
  // name = "Bob";  // ERROR — never do this
  return <p>{name}</p>;
}
```

---

**2. What is the difference between passing a string prop and a non-string prop?**

String props can use regular HTML-style quotes. All other values — numbers, booleans, objects, arrays, and expressions — must use curly braces.

```jsx
<Card
  title="Wireless Mouse"     // string — quotes OK
  price={29}                 // number — curly braces
  active={true}              // boolean — curly braces
  style={{ color: "red" }}   // object — curly braces (double braces: outer = JS, inner = object)
/>
```

---

**3. How do default props work, and when do defaults NOT apply?**

Default values are set via ES6 default parameter syntax. They apply only when a prop is `undefined`. If the parent explicitly passes `null` or `false`, the default does not activate.

```jsx
function Button({ label = "Click Me", disabled = false }) {
  return <button disabled={disabled}>{label}</button>;
}

<Button />                        // label = "Click Me", disabled = false
<Button label={undefined} />      // still uses default "Click Me"
<Button label={null} />           // label = null  (default does NOT apply)
```

---

**4. How do you pass a function as a prop and why would you?**

Pass the function reference (not the invocation) as an attribute. This is called the callback pattern — it lets child components communicate upward to their parent without owning the logic themselves.

```jsx
function App() {
  function handleDelete(id) {
    console.log("Deleting:", id);
  }
  return <ItemCard id={5} onDelete={handleDelete} />;
}

function ItemCard({ id, onDelete }) {
  return <button onClick={() => onDelete(id)}>Delete</button>;
}

// Click → console: "Deleting: 5"
```

---

**5. What is the `children` prop and how is it different from regular props?**

`children` is a special built-in prop that captures whatever JSX is placed between a component's opening and closing tags. Unlike regular props that are passed as attributes, children come from the component's inner content.

```jsx
function Panel({ title, children }) {
  return (
    <div>
      <h3>{title}</h3>
      {children}
    </div>
  );
}

<Panel title="Info">
  <p>This is the body content.</p>   {/* becomes children */}
  <button>OK</button>
</Panel>
```

---

**6. Can you pass JSX as a prop?**

Yes. Any valid JavaScript value can be a prop, including JSX elements. This is useful for slot-like patterns where the parent injects entire sections of UI.

```jsx
function Layout({ header, sidebar, children }) {
  return (
    <div>
      <header>{header}</header>
      <aside>{sidebar}</aside>
      <main>{children}</main>
    </div>
  );
}

<Layout
  header={<Navbar user={currentUser} />}
  sidebar={<SideMenu links={navLinks} />}
>
  <DashboardContent />
</Layout>
```

---

**7. What is prop drilling and how do you fix it?**

Prop drilling is when data is passed through intermediate components that do not use it — they only forward it deeper. It couples those intermediate components to data they do not care about.

```
App (user data)
  → Layout (forwards user, never uses it)
    → Sidebar (forwards user, never uses it)
      → Avatar (actually uses user)
```

Solutions:
- **Context API** — broadcast data to any component in the tree without threading props.
- **Redux / Zustand** — external global store.
- **Component composition** — lift the consumer up closer to the data.

---

**8. What does PropTypes do and why prefer TypeScript over it?**

`PropTypes` is a runtime check that logs warnings in the browser console if wrong types are passed in development. TypeScript is preferred because it catches type errors at compile time, before the code runs, and provides autocomplete in editors.

```jsx
import PropTypes from "prop-types";

Card.propTypes = {
  title:   PropTypes.string.isRequired,
  price:   PropTypes.number,
  onClick: PropTypes.func,
};

// TypeScript alternative:
interface CardProps {
  title: string;
  price?: number;
  onClick?: () => void;
}
function Card({ title, price, onClick }: CardProps) { ... }
```

---

**9. What happens if a required prop is not passed?**

The component receives `undefined` for that prop. If `PropTypes.isRequired` is declared, a warning appears in the console during development. The component still renders — PropTypes never throws errors.

```
Warning: Failed prop type: The prop `title` is marked as required
in `Card`, but its value is `undefined`.
```

---

**10. How does lifting state up relate to props?**

When sibling components need to share data, that data is moved to their closest common parent. The parent owns the state and passes it *down* as props. When a child needs to modify it, the parent passes a callback *down* as a prop too. This keeps a single source of truth.

```jsx
function App() {
  const [text, setText] = React.useState("");
  return (
    <>
      <InputBox value={text} onChange={setText} />
      <Preview value={text} />
    </>
  );
}

function InputBox({ value, onChange }) {
  return <input value={value} onChange={e => onChange(e.target.value)} />;
}

function Preview({ value }) {
  return <p>Preview: {value}</p>;
}
```
