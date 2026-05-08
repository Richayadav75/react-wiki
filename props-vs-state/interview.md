# Props vs State Interview Questions

---

**1. What is the core difference between props and state?**

Props are *external* data passed into a component by its parent — the component cannot change them. State is *internal* data managed by the component itself — the component can change it at any time via the setter function.

```jsx
// Props — parent sets these, child reads them
function Greeting({ name }) {
  return <h1>Hello, {name}</h1>;
}

// State — component sets and updates this itself
function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

---

**2. Can a child component modify its own props?**

No. Props are read-only (immutable). If a child needs to signal a change, the parent passes a callback function as a prop, and the child calls it. The parent then updates its own state, which flows down as new prop values.

```jsx
// WRONG — never do this
function Child({ name }) {
  name = "Bob"; // illegal mutation — React does not allow this
}

// CORRECT — signal via callback
function Child({ name, onChangeName }) {
  return <button onClick={() => onChangeName("Bob")}>{name}</button>;
}
```

---

**3. Both props and state can trigger a re-render. What is the difference?**

- State change: the component that owns the state re-renders (triggered by calling the setter).
- Prop change: a child re-renders because its parent re-rendered and passed new prop values. The child has no control over this.

```
State change:  Counter calls setCount() → Counter re-renders
Prop change:   App re-renders → passes new `name` to Greeting → Greeting re-renders
               (Greeting did not initiate this — App did)
```

---

**4. What is the most common mistake involving props and state?**

Copying a prop value into state as the initial value and then treating the child's copy as the source of truth. If the parent's value updates later, the child's state stays at the original value — they diverge.

```jsx
// WRONG — child state gets out of sync with parent prop
function UserLabel({ name }) {
  const [displayName, setDisplayName] = useState(name); // captured once
  // If parent changes name prop, displayName is still the old value
  return <span>{displayName}</span>;
}

// CORRECT — read the prop directly
function UserLabel({ name }) {
  return <span>{name}</span>; // always shows current prop
}
```

---

**5. What is "single source of truth" in React?**

It means each piece of data should exist in exactly one place. If multiple components display the same value, that value should live in their common parent as state and flow down as props. Storing the same data in two places creates the risk of them getting out of sync.

```jsx
// GOOD — one source of truth in App
function App() {
  const [count, setCount] = useState(0);
  return (
    <>
      <DisplayA count={count} />
      <DisplayB count={count} />
      <IncrementButton onClick={() => setCount(c => c + 1)} />
    </>
  );
}
// DisplayA and DisplayB always agree because they both read from App's state.
```

---

**6. What is the difference between a controlled and uncontrolled component?**

A controlled component has its input value driven by React state. Every keystroke goes through `onChange → setState → re-render → value={state}`. An uncontrolled component lets the DOM manage the value, accessed via `ref` on demand.

```jsx
// Controlled — React owns the value
function Controlled() {
  const [val, setVal] = useState("");
  return <input value={val} onChange={e => setVal(e.target.value)} />;
}

// Uncontrolled — DOM owns the value
function Uncontrolled() {
  const ref = useRef(null);
  return <input ref={ref} defaultValue="" />;
  // read with ref.current.value when needed
}
```

---

**7. When should you lift state up?**

Lift state up when two or more sibling components need to access or update the same piece of data. Move the state to their closest common ancestor, and pass it down as props.

```jsx
// Both SearchInput and ResultsList need the same query
function App() {
  const [query, setQuery] = useState("");
  return (
    <>
      <SearchInput value={query} onChange={setQuery} />
      <ResultsList query={query} />
    </>
  );
}
```

---

**8. Is state shared between two instances of the same component?**

No. Each instance of a component has completely independent state. Rendering `<Counter />` twice gives you two separate counters.

```jsx
function App() {
  return (
    <>
      <Counter />  {/* count starts at 0, independent */}
      <Counter />  {/* count starts at 0, independent */}
    </>
  );
}
// Clicking +1 on the first counter does NOT affect the second.
```

---

**9. How do you pass data from a child back up to its parent?**

The parent defines a function that updates its state. It passes that function as a prop to the child. When the child calls the prop function, the parent's state updates.

```jsx
function Parent() {
  const [selected, setSelected] = useState(null);

  return (
    <>
      <ChildSelector onSelect={setSelected} />
      <p>Selected: {selected ?? "none"}</p>
    </>
  );
}

function ChildSelector({ onSelect }) {
  return (
    <>
      <button onClick={() => onSelect("A")}>Choose A</button>
      <button onClick={() => onSelect("B")}>Choose B</button>
    </>
  );
}
```

---

**10. Give a real-world scenario where you would use props vs state.**

A product listing page: the product data (name, price, image) is fetched by the parent and passed down as props to `<ProductCard />` — the card just displays it. The "Add to Cart" button toggles a local `added` boolean, which is state inside `<ProductCard />` because it is specific to that card's interaction.

```jsx
function ProductCard({ name, price, imageUrl }) {
  // Props: name, price, imageUrl — given by parent, read-only
  // State: added — owned by this card, changes on button click
  const [added, setAdded] = useState(false);

  return (
    <div>
      <img src={imageUrl} alt={name} />
      <h3>{name}</h3>
      <p>${price}</p>
      <button onClick={() => setAdded(true)} disabled={added}>
        {added ? "Added to Cart" : "Add to Cart"}
      </button>
    </div>
  );
}
```
