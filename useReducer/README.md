- Category: State Management
- Difficulty: Intermediate
- Related: useState, useContext, Redux

`useReducer` is an alternative to `useState` for **complex state logic** — particularly when the next state depends on the previous state, or when multiple sub-values are updated together. It follows the Redux reducer pattern.

## Syntax

```tsx
const [state, dispatch] = useReducer(reducer, initialState);
```

- `reducer` — a pure function `(state, action) => newState`
- `initialState` — the starting state value
- `dispatch` — call it with an action to trigger a state update

## Example — Shopping cart

```tsx
import { useReducer } from 'react';

type CartItem = { id: string; name: string; qty: number };

type State = { items: CartItem[] };

type Action =
  | { type: 'ADD'; item: CartItem }
  | { type: 'REMOVE'; id: string }
  | { type: 'CLEAR' };

function cartReducer(state: State, action: Action): State {
  switch (action.type) {
    case 'ADD':
      const existing = state.items.find(i => i.id === action.item.id);
      if (existing) {
        return {
          items: state.items.map(i =>
            i.id === action.item.id ? { ...i, qty: i.qty + 1 } : i
          ),
        };
      }
      return { items: [...state.items, action.item] };
    case 'REMOVE':
      return { items: state.items.filter(i => i.id !== action.id) };
    case 'CLEAR':
      return { items: [] };
    default:
      return state;
  }
}

function Cart() {
  const [cart, dispatch] = useReducer(cartReducer, { items: [] });

  return (
    <>
      {cart.items.map(item => (
        <div key={item.id}>
          {item.name} x {item.qty}
          <button onClick={() => dispatch({ type: 'REMOVE', id: item.id })}>Remove</button>
        </div>
      ))}
      <button onClick={() => dispatch({ type: 'CLEAR' })}>Clear cart</button>
    </>
  );
}
```

## useState vs useReducer

| Scenario | Prefer |
|----------|--------|
| 1–2 primitive values | `useState` |
| Related fields updated together | `useReducer` |
| Next state depends on previous | `useReducer` |
| Business logic shared across components | `useReducer` + Context |
| Complex state machine | `useReducer` |

## Common Pitfalls

- **Mutating state in the reducer** — Always return a new object/array; never mutate the existing state.
- **Side effects in the reducer** — Reducers must be pure functions; put side effects in `useEffect`.
- **Over-engineering simple state** — If you have one boolean flag, `useState` is cleaner.

## Practice
```tsx
const initialState = { count: 0 };

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    default:
      throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <div className="p-4 border rounded">
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
    </div>
  );
}
```

## Interview Questions
1. **How is useReducer different from useState?**
   `useReducer` is better for complex state that has multiple sub-values or when the next state depends on the previous one. It also helps decouple the *what* happens (action) from the *how* it's updated (reducer).
2. **Can a reducer be asynchronous?**
   No, reducers must be pure functions. They should only calculate the next state. Any async logic (like API calls) should happen outside the reducer, usually in an event handler or `useEffect`, which then dispatches an action.

## Learn More
- [React Docs — useReducer](https://react.dev/reference/react/useReducer)
- [Extracting State Logic into a Reducer](https://react.dev/learn/extracting-state-logic-into-a-reducer)
