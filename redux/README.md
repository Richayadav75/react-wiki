- Category: State Management
- Difficulty: Advanced
- Related: props-vs-state, context-api, useReducer, props-drilling

### Redux — Predictable Global State Management
Redux is a pattern and library for managing application state in a single, centralized store. Any component can read from or write to this store without prop drilling. The key guarantee: state can only change in one specific, predictable way — through dispatched actions processed by pure reducer functions.

**Analogy**
Redux is like a bank. Your money (state) is stored in the bank vault (store) — not in your wallet (component). To deposit or withdraw (change state), you fill out a form (action) and hand it to a teller (reducer). The teller follows strict rules (pure function) to update the vault. You never reach directly into the vault yourself. Every transaction is logged (Redux DevTools).

---

### 1. The Three Pillars — Store, Action, Reducer

**Theory**: Redux has exactly three moving parts. Understanding each one is the entire mental model.

**Working Flow**
![flow-chart](flow-chart.png)

**The Store**
```javascript
import { configureStore } from "@reduxjs/toolkit";
import cartReducer from "./cartSlice";

const store = configureStore({
  reducer: {
    cart: cartReducer,   // state.cart is managed by cartReducer
  },
});

// store.getState()    → { cart: { items: [], total: 0 } }
// store.dispatch(action) → triggers reducer
// store.subscribe(fn)    → called after every state change
```

**An Action**
```javascript
// Actions are plain objects — must have a "type" field
const addItem     = { type: "cart/addItem",    payload: { id: 1, name: "Book", price: 15 } };
const removeItem  = { type: "cart/removeItem", payload: { id: 1 } };
const clearCart   = { type: "cart/clear" };
```

**A Reducer**
```javascript
// Pure function — no mutations, no API calls, no randomness
function cartReducer(state = { items: [], total: 0 }, action) {
  switch (action.type) {
    case "cart/addItem":
      return {
        ...state,
        items: [...state.items, action.payload],
        total: state.total + action.payload.price,
      };
    case "cart/removeItem":
      const filtered = state.items.filter(i => i.id !== action.payload.id);
      return {
        ...state,
        items: filtered,
        total: filtered.reduce((sum, i) => sum + i.price, 0),
      };
    case "cart/clear":
      return { items: [], total: 0 };
    default:
      return state;  // always return current state for unknown actions
  }
}
```

**Output**
```
Initial state: { items: [], total: 0 }

dispatch(addItem Book $15):  { items: [{id:1, name:"Book", price:15}], total: 15 }
dispatch(addItem Pen $3):    { items: [{...Book}, {id:2, name:"Pen", price:3}], total: 18 }
dispatch(removeItem id:1):   { items: [{id:2, name:"Pen", price:3}], total: 3 }
dispatch(clearCart):         { items: [], total: 0 }
```

---

### 2. The Full Data Flow — UI Event to Re-render

**Theory**: Redux enforces a strict unidirectional data flow. Nothing happens outside this loop.

**Working Flow**
![flow-chart-2](flow-chart-2.png)

**Example — connecting the loop**
```jsx
// In a component
import { useDispatch, useSelector } from "react-redux";

function ProductCard({ product }) {
  const dispatch  = useDispatch();
  const cartCount = useSelector(state => state.cart.items.length);

  const handleAdd = () => {
    dispatch({ type: "cart/addItem", payload: product });
    // ↑ This single line triggers the entire Redux loop
  };

  return (
    <div>
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      <button onClick={handleAdd}>Add to Cart ({cartCount})</button>
    </div>
  );
}
```

**Output**
```
Before click: "Add to Cart (2)"
After click:  "Add to Cart (3)"  ← re-renders because useSelector sees new state
```

---

### 3. Redux Toolkit — The Modern Way

**Theory**: Vanilla Redux requires a lot of boilerplate — separate action type constants, action creator functions, and switch statements in reducers. Redux Toolkit (RTK) collapses all of this into a single `createSlice` call. It also uses Immer internally, so you can write "mutating" code that is actually safe.

**Working Flow**
![flow-chart-3](flow-chart-3.png)

**Cart Slice — full example**
```javascript
// store/cartSlice.js
import { createSlice } from "@reduxjs/toolkit";

const cartSlice = createSlice({
  name: "cart",
  initialState: {
    items: [],
    total: 0,
  },
  reducers: {
    addItem(state, action) {
      const existing = state.items.find(i => i.id === action.payload.id);
      if (existing) {
        existing.quantity += 1;
      } else {
        state.items.push({ ...action.payload, quantity: 1 });
      }
      state.total += action.payload.price;
    },
    removeItem(state, action) {
      const item = state.items.find(i => i.id === action.payload);
      if (item) state.total -= item.price * item.quantity;
      state.items = state.items.filter(i => i.id !== action.payload);
    },
    clearCart(state) {
      state.items = [];
      state.total = 0;
    },
  },
});

export const { addItem, removeItem, clearCart } = cartSlice.actions;
export default cartSlice.reducer;
```

**Store setup**
```javascript
// store/index.js
import { configureStore } from "@reduxjs/toolkit";
import cartReducer from "./cartSlice";
import userReducer from "./userSlice";

export const store = configureStore({
  reducer: {
    cart: cartReducer,
    user: userReducer,
  },
});
```

**Provide store to app**
```jsx
// index.js
import { Provider } from "react-redux";
import { store } from "./store";

root.render(
  <Provider store={store}>
    <App />
  </Provider>
);
```

---

### 4. useSelector and useDispatch — Reading and Writing

**Theory**: `useSelector` subscribes a component to a slice of state. `useDispatch` gives a component the ability to send actions. These two hooks replace the older `connect()` HOC pattern.

**Working Flow**
![flow-chart-4](flow-chart-4.png)

**Example — Cart UI**
```jsx
import { useSelector, useDispatch } from "react-redux";
import { addItem, removeItem, clearCart } from "./store/cartSlice";

function CartSidebar() {
  const items    = useSelector(state => state.cart.items);
  const total    = useSelector(state => state.cart.total);
  const dispatch = useDispatch();

  return (
    <div>
      <h2>Cart ({items.length} items)</h2>
      {items.map(item => (
        <div key={item.id}>
          <span>{item.name} x{item.quantity}</span>
          <button onClick={() => dispatch(removeItem(item.id))}>Remove</button>
        </div>
      ))}
      <p>Total: ${total.toFixed(2)}</p>
      <button onClick={() => dispatch(clearCart())}>Clear Cart</button>
    </div>
  );
}
```

**Output**
```
Cart (2 items)
  Book x1    [Remove]
  Pen  x3    [Remove]
Total: $24.00
[Clear Cart]

After clicking "Clear Cart":
Cart (0 items)
Total: $0.00
```

---

### 5. Selectors — Derived State and Memoization

**Theory**: A selector is a function that computes derived data from the store. Placing selector logic outside components keeps components thin. `createSelector` from `reselect` memoizes expensive derivations so they only recompute when inputs change.

**Working Flow**
![flow-chart-5](flow-chart-5.png)

**Example**
```javascript
// store/cartSelectors.js
import { createSelector } from "@reduxjs/toolkit";

const selectCartItems = state => state.cart.items;

export const selectItemCount = createSelector(
  selectCartItems,
  items => items.reduce((n, item) => n + item.quantity, 0)
);

export const selectCartTotal = createSelector(
  selectCartItems,
  items => items.reduce((sum, item) => sum + item.price * item.quantity, 0)
);

// In component
function CartBadge() {
  const count = useSelector(selectItemCount);
  return <span>{count}</span>; // re-renders only when item count changes
}
```

---

### 6. connect() vs useSelector/useDispatch

**Theory**: Before hooks, components connected to Redux using the `connect()` HOC. Modern React uses hooks. Both work, but hooks produce cleaner, more readable code.

**Working Flow**
![flow-chart-6](flow-chart-6.png)

**Side-by-side comparison**
```jsx
// OLD: connect() pattern
import { connect } from "react-redux";

function CartCount({ itemCount, onClear }) {
  return <button onClick={onClear}>{itemCount} items</button>;
}

const mapStateToProps    = state => ({ itemCount: state.cart.items.length });
const mapDispatchToProps = dispatch => ({ onClear: () => dispatch(clearCart()) });

export default connect(mapStateToProps, mapDispatchToProps)(CartCount);

// NEW: hooks pattern (same behavior, much less code)
function CartCount() {
  const itemCount = useSelector(state => state.cart.items.length);
  const dispatch  = useDispatch();
  return <button onClick={() => dispatch(clearCart())}>{itemCount} items</button>;
}
```

---

### Real-World Example — E-Commerce Cart

```text
State shape:
{
  cart:    { items: [], total: 0 },
  user:    { name: "Alice", isLoggedIn: true },
  ui:      { sidebarOpen: false, theme: "light" },
  orders:  { history: [], loading: false }
}

Slices:
  cartSlice    → addItem, removeItem, updateQty, clearCart
  userSlice    → login, logout, updateProfile
  uiSlice      → toggleSidebar, setTheme
  orderSlice   → fetchOrders (async with createAsyncThunk)

Data flow example — checkout:
  1. User clicks "Checkout"
  2. dispatch(createOrder(cart.items))
  3. orderSlice async thunk: POST /api/orders
  4. On success: dispatch(clearCart()) + dispatch(addToHistory(order))
  5. Navigate to confirmation page
```

---

### When to use Redux vs Context API

```text
Use Redux when:                          Use Context when:
──────────────────────────────           ──────────────────────────────
Many components share state              Few components share state
State updates are frequent               State changes rarely
State transitions are complex            Simple toggle/theme/auth flag
You need DevTools / time-travel          No debugging tooling needed
Team of 3+ engineers                     Solo or small project
Multiple slices of global state          1-2 pieces of global data
```

---

[View Interview Questions](./interview.md)
