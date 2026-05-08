- Category: React Hooks
- Difficulty: Intermediate
- Related: useState, useContext, redux, hooks

### useReducer — Manage Complex State with a Reducer Function
`useReducer` is a React hook that manages state through a pure **reducer function** — it takes the current state and an action, then returns the next state. It is an alternative to `useState` when state logic has multiple cases or multiple sub-values that change together.

**Analogy**
A bank teller. You don't reach into the vault yourself (mutate state directly). You hand the teller a written slip (action). The teller checks the current balance and your slip, applies the transaction, and returns the new balance (new state). The reducer is the teller — all state changes flow through it.

---

### 1. Anatomy — State, Action, Reducer, Dispatch

**Theory**
`useReducer` has four moving parts:
- **state** — current value (like useState)
- **action** — plain object with a `type` field describing what happened
- **reducer** — pure function: `(state, action) => newState`
- **dispatch** — function you call to send an action to the reducer

`const [state, dispatch] = useReducer(reducer, initialState)`

**Working Flow**

![flow-chart](flow-chart.png)

**Example**
```jsx
import { useReducer } from 'react';

// Step 1 — Define the reducer (all update rules live here)
function counterReducer(state, action) {
  switch (action.type) {
    case 'INCREMENT': return { count: state.count + 1 };
    case 'DECREMENT': return { count: state.count - 1 };
    case 'RESET':     return { count: 0 };
    case 'SET':       return { count: action.payload };
    default:          return state;
  }
}

// Step 2 — Use in component
function Counter() {
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+1</button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-1</button>
      <button onClick={() => dispatch({ type: 'RESET' })}>Reset</button>
      <button onClick={() => dispatch({ type: 'SET', payload: 10 })}>
        Set to 10
      </button>
    </div>
  );
}
```

**Output**
```
Initial render  → Count: 0
+1 click        → Count: 1
+1 click        → Count: 2
Reset           → Count: 0
Set to 10       → Count: 10
```

**Explanation**
Each button dispatches an action `{ type: '...' }`. The reducer receives current state + action, returns a new state object. React re-renders with the new state. The component never touches state directly — all logic lives in the reducer and is easy to test.

---

### 2. Complex State — Todo List with Multiple Action Types

**Theory**
`useReducer` shines when state has many update types. Instead of multiple `useState` calls with complex setter logic scattered across the component, all transitions live in one readable reducer function.

**Working Flow**

![flow-chart-2](flow-chart-2.png)

**Example**
```jsx
import { useReducer, useState } from 'react';

const initialState = {
  todos: [],
  filter: 'all',
};

function todoReducer(state, action) {
  switch (action.type) {
    case 'ADD':
      return {
        ...state,
        todos: [
          ...state.todos,
          { id: Date.now(), text: action.payload, done: false },
        ],
      };
    case 'TOGGLE':
      return {
        ...state,
        todos: state.todos.map(t =>
          t.id === action.payload ? { ...t, done: !t.done } : t
        ),
      };
    case 'DELETE':
      return {
        ...state,
        todos: state.todos.filter(t => t.id !== action.payload),
      };
    case 'SET_FILTER':
      return { ...state, filter: action.payload };
    default:
      return state;
  }
}

function TodoApp() {
  const [state, dispatch] = useReducer(todoReducer, initialState);
  const [input, setInput] = useState('');

  const visible = state.todos.filter(t => {
    if (state.filter === 'done')   return t.done;
    if (state.filter === 'active') return !t.done;
    return true;
  });

  return (
    <div>
      <input
        value={input}
        onChange={e => setInput(e.target.value)}
        placeholder="New todo..."
      />
      <button onClick={() => {
        dispatch({ type: 'ADD', payload: input });
        setInput('');
      }}>
        Add
      </button>

      <div>
        {['all', 'active', 'done'].map(f => (
          <button
            key={f}
            onClick={() => dispatch({ type: 'SET_FILTER', payload: f })}
          >
            {f}
          </button>
        ))}
      </div>

      <ul>
        {visible.map(t => (
          <li key={t.id}>
            <input
              type="checkbox"
              checked={t.done}
              onChange={() => dispatch({ type: 'TOGGLE', payload: t.id })}
            />
            <span
              style={{ textDecoration: t.done ? 'line-through' : 'none' }}
            >
              {t.text}
            </span>
            <button onClick={() => dispatch({ type: 'DELETE', payload: t.id })}>
              ✕
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

**Output**
```
Add "Buy milk"  → todos: [{ text:"Buy milk", done:false }]
Add "Code"      → todos: [..., { text:"Code", done:false }]
Toggle id:1     → todos: [{ done:true }, { done:false }]
Filter "done"   → shows only completed todos
Delete id:1     → removes it from list
```

---

### 3. useReducer + useContext — Mini Redux

**Theory**
Combine `useReducer` for state logic with `useContext` for distribution. State and dispatch are provided at the top level, consumed anywhere in the tree — global state without any library.

**Working Flow**

![flow-chart-3](flow-chart-3.png)

**Example**
```jsx
import { useReducer, useContext, createContext } from 'react';

const StateCtx    = createContext(null);
const DispatchCtx = createContext(null);

function cartReducer(state, action) {
  switch (action.type) {
    case 'ADD':    return [...state, action.payload];
    case 'REMOVE': return state.filter(item => item.id !== action.payload);
    case 'CLEAR':  return [];
    default:       return state;
  }
}

// Provider wraps the app
function CartProvider({ children }) {
  const [cart, dispatch] = useReducer(cartReducer, []);
  return (
    <StateCtx.Provider value={cart}>
      <DispatchCtx.Provider value={dispatch}>
        {children}
      </DispatchCtx.Provider>
    </StateCtx.Provider>
  );
}

// Custom hooks for clean access
const useCart         = () => useContext(StateCtx);
const useCartDispatch = () => useContext(DispatchCtx);

// Any component can read state or dispatch
function CartCount() {
  const cart = useCart();
  return <span>Cart: {cart.length} items</span>;
}

function AddButton({ product }) {
  const dispatch = useCartDispatch();
  return (
    <button onClick={() => dispatch({ type: 'ADD', payload: product })}>
      Add to Cart
    </button>
  );
}
```

---

### 4. useState vs useReducer — Decision Guide

**Theory**
Both manage state — the choice depends on complexity.

**Working Flow**

![flow-chart-4](flow-chart-4.png)

**Example**
```jsx
// ✅ useState — simple, independent values
const [name, setName]         = useState('');
const [email, setEmail]       = useState('');
const [isLoading, setLoading] = useState(false);

// ✅ useReducer — interrelated state changing together
const [state, dispatch] = useReducer(formReducer, {
  name: '',
  email: '',
  isLoading: false,
  error: null,
  success: false,
});
// One dispatch('SUBMIT') sets isLoading=true, error=null, success=false atomically
```

**Output**
```
Situation                          → Use
Single value (counter, flag)       → useState
2-3 independent values             → useState
Object with 4+ interrelated fields → useReducer
Multiple fields change together    → useReducer
State logic needs unit tests       → useReducer
```

---

### 5. Real-World — Shopping Cart

**Working Flow**

![flow-chart-5](flow-chart-5.png)

**Example**
```jsx
import { useReducer } from 'react';

const cartReducer = (state, action) => {
  switch (action.type) {
    case 'ADD_ITEM': {
      const exists = state.items.find(i => i.id === action.payload.id);
      return {
        ...state,
        items: exists
          ? state.items.map(i =>
              i.id === action.payload.id
                ? { ...i, qty: i.qty + 1 }
                : i
            )
          : [...state.items, { ...action.payload, qty: 1 }],
      };
    }
    case 'REMOVE_ITEM':
      return {
        ...state,
        items: state.items.filter(i => i.id !== action.payload),
      };
    case 'UPDATE_QTY':
      return {
        ...state,
        items: state.items.map(i =>
          i.id === action.payload.id
            ? { ...i, qty: action.payload.qty }
            : i
        ),
      };
    case 'CLEAR_CART':
      return { items: [] };
    default:
      return state;
  }
};

function Cart() {
  const [cart, dispatch] = useReducer(cartReducer, { items: [] });

  const total = cart.items.reduce(
    (sum, i) => sum + i.price * i.qty,
    0
  );

  return (
    <div>
      {cart.items.map(item => (
        <div key={item.id}>
          <span>{item.name} × {item.qty} = ₹{item.price * item.qty}</span>
          <button onClick={() =>
            dispatch({
              type: 'UPDATE_QTY',
              payload: { id: item.id, qty: item.qty + 1 },
            })
          }>
            +
          </button>
          <button onClick={() =>
            dispatch({ type: 'REMOVE_ITEM', payload: item.id })
          }>
            Remove
          </button>
        </div>
      ))}
      <p>Total: ₹{total}</p>
      <button onClick={() => dispatch({ type: 'CLEAR_CART' })}>
        Clear Cart
      </button>
    </div>
  );
}
```

---

[View Interview Questions](./interview.md)
