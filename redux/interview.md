# Redux — Interview Questions

---

**1. What are the three core principles of Redux?**

- **Single source of truth**: The entire app state lives in one store object. Any component can read any piece of it.
- **State is read-only**: You cannot modify state directly. The only way to change it is to dispatch an action.
- **Changes are made with pure functions**: Reducers take (state, action) and return a new state — no mutations, no API calls, no randomness.

```javascript
// You cannot do this:
store.getState().cart.items.push(product); // WRONG — direct mutation

// You must do this:
store.dispatch({ type: "cart/addItem", payload: product }); // CORRECT
```

---

**2. What is the difference between an Action and an Action Creator?**

An **action** is the plain object itself. An **action creator** is a function that creates and returns that object. Redux Toolkit's `createSlice` auto-generates action creators.

```javascript
// Action (the plain object)
const action = { type: "cart/addItem", payload: { id: 1, name: "Book", price: 15 } };

// Action creator (function that makes the object)
function addItem(product) {
  return { type: "cart/addItem", payload: product };
}

// RTK auto-generates action creators from createSlice
const { addItem, removeItem } = cartSlice.actions;
dispatch(addItem({ id: 1, name: "Book", price: 15 }));
// → dispatches { type: "cart/addItem", payload: { id:1, name:"Book", price:15 } }
```

---

**3. What is a Reducer and why must it be a pure function?**

A reducer is a function that takes `(state, action)` and returns new state. It must be pure because Redux compares state by reference — if you mutate the existing object, React won't see a change and won't re-render.

```javascript
// WRONG — mutating state directly (pure function violation)
function cartReducer(state = { items: [] }, action) {
  if (action.type === "cart/addItem") {
    state.items.push(action.payload); // mutates! Redux won't detect change
    return state;
  }
  return state;
}

// CORRECT — returning a new object
function cartReducer(state = { items: [] }, action) {
  if (action.type === "cart/addItem") {
    return { ...state, items: [...state.items, action.payload] }; // new object
  }
  return state;
}

// RTK with Immer — "mutation" syntax that is actually safe
const cartSlice = createSlice({
  name: "cart",
  initialState: { items: [] },
  reducers: {
    addItem(state, action) {
      state.items.push(action.payload); // Immer converts this to a new object
    },
  },
});
```

---

**4. What is Redux Toolkit and why was it created?**

Redux Toolkit (RTK) is the official recommended way to write Redux. It was created to solve the most common complaints about Redux: too much boilerplate, confusing setup, and accidental state mutations.

```javascript
// WITHOUT RTK — 4 separate files for one feature
// actionTypes.js, actions.js, reducer.js, store.js = ~60 lines

// WITH RTK — one file, ~20 lines
import { createSlice, configureStore } from "@reduxjs/toolkit";

const counterSlice = createSlice({
  name: "counter",
  initialState: { value: 0 },
  reducers: {
    increment: state => { state.value += 1; },
    decrement: state => { state.value -= 1; },
    addBy:     (state, action) => { state.value += action.payload; },
  },
});

export const { increment, decrement, addBy } = counterSlice.actions;

const store = configureStore({ reducer: { counter: counterSlice.reducer } });
```

---

**5. What is the difference between connect() and useSelector/useDispatch?**

`connect()` is a HOC pattern from older Redux. `useSelector`/`useDispatch` are hooks introduced in React-Redux v7.1. Hooks are simpler and eliminate the need for `mapStateToProps` and `mapDispatchToProps`.

```jsx
// OLD — connect() HOC
const mapStateToProps    = state => ({ count: state.counter.value });
const mapDispatchToProps = dispatch => ({ increment: () => dispatch(increment()) });

export default connect(mapStateToProps, mapDispatchToProps)(Counter);

// NEW — hooks (same behavior, much less code)
function Counter() {
  const count    = useSelector(state => state.counter.value);
  const dispatch = useDispatch();

  return (
    <div>
      <p>{count}</p>
      <button onClick={() => dispatch(increment())}>+</button>
    </div>
  );
}
```

---

**6. When should you use Redux vs the Context API?**

```text
Context API is better when:             Redux is better when:
─────────────────────────────           ─────────────────────────────
Sharing theme, locale, auth flag        Complex app-wide state (cart, orders)
State changes rarely                    State changes frequently
Small to medium app                     Large team / large app
No DevTools needed                      Need time-travel debugging
1-2 global values                       Many slices of state
```

```jsx
// Context — simple auth flag
const AuthContext = createContext();
function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  return <AuthContext.Provider value={{ user, setUser }}>{children}</AuthContext.Provider>;
}

// Redux — complex cart with quantities, totals, syncing
const cartSlice = createSlice({ /* addItem, removeItem, updateQty, checkout... */ });
```

---

**7. What is middleware in Redux and what is it used for?**

Middleware sits between `dispatch` and the reducer. It intercepts actions before they reach the reducer, enabling logging, crash reporting, and async operations.

```javascript
// Redux Thunk (built into RTK) — allows dispatching async functions
export const fetchOrders = createAsyncThunk(
  "orders/fetchAll",
  async (userId) => {
    const response = await fetch(`/api/users/${userId}/orders`);
    return response.json(); // returned value becomes action.payload
  }
);

// In component
const dispatch = useDispatch();
useEffect(() => {
  dispatch(fetchOrders(userId));
}, [userId]);

// In slice — handle async states
extraReducers: (builder) => {
  builder
    .addCase(fetchOrders.pending,   state => { state.loading = true; })
    .addCase(fetchOrders.fulfilled, (state, action) => {
      state.loading = false;
      state.history = action.payload;
    })
    .addCase(fetchOrders.rejected,  state => { state.loading = false; state.error = true; });
}
```

---

**8. What is a selector and why use createSelector from reselect?**

A selector is a function that extracts and derives data from the store. `createSelector` memoizes the result — it only recomputes when the input values change, preventing unnecessary re-renders.

```javascript
import { createSelector } from "@reduxjs/toolkit";

// Basic selector (no memoization — recomputes every render)
const selectItems = state => state.cart.items;

// Memoized selector — only recalculates when state.cart.items changes
const selectCartTotal = createSelector(
  selectItems,
  items => items.reduce((sum, item) => sum + item.price * item.quantity, 0)
);

const selectItemCount = createSelector(
  selectItems,
  items => items.reduce((n, item) => n + item.quantity, 0)
);

// In component
function CartSummary() {
  const total = useSelector(selectCartTotal); // memoized
  const count = useSelector(selectItemCount); // memoized
  return <p>{count} items — ${total.toFixed(2)}</p>;
}
```

---

**9. What happens when you dispatch an action that no reducer handles?**

The action passes through every reducer, each sees it, none match, and each returns the current state unchanged. The store state does not change and no re-renders occur.

```javascript
// Action type no reducer knows about
store.dispatch({ type: "UNKNOWN_ACTION" });

// Each reducer hits the default case:
function cartReducer(state = initialState, action) {
  switch (action.type) {
    // ... no match ...
    default: return state; // returns unchanged state
  }
}

// Result: store.getState() === same object as before → no re-render
```

---

**10. How do you handle async operations in Redux Toolkit?**

Use `createAsyncThunk`. It auto-generates three action types: `pending`, `fulfilled`, and `rejected`, which you handle in `extraReducers`.

```javascript
// productSlice.js
const fetchProducts = createAsyncThunk("products/fetchAll", async (category) => {
  const res = await fetch(`/api/products?category=${category}`);
  if (!res.ok) throw new Error("Failed to fetch");
  return res.json();
});

const productSlice = createSlice({
  name: "products",
  initialState: { items: [], loading: false, error: null },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchProducts.pending,   state => {
        state.loading = true;
        state.error   = null;
      })
      .addCase(fetchProducts.fulfilled, (state, action) => {
        state.loading = false;
        state.items   = action.payload;
      })
      .addCase(fetchProducts.rejected,  (state, action) => {
        state.loading = false;
        state.error   = action.error.message;
      });
  },
});

// In component
function ProductList({ category }) {
  const dispatch  = useDispatch();
  const { items, loading, error } = useSelector(state => state.products);

  useEffect(() => { dispatch(fetchProducts(category)); }, [category]);

  if (loading) return <p>Loading...</p>;
  if (error)   return <p>Error: {error}</p>;
  return <ul>{items.map(p => <li key={p.id}>{p.name}</li>)}</ul>;
}
```
