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

## Learn More

- [React Docs — useReducer](https://react.dev/reference/react/useReducer)
- [Extracting State Logic into a Reducer](https://react.dev/learn/extracting-state-logic-into-a-reducer)
