- Category: State Management
- Difficulty: Intermediate
- Related: useState, useReducer, Zustand

`useContext` lets you read and subscribe to a **Context** — a way to share values (theme, user, locale, etc.) across the component tree without prop-drilling.

## Setup in 3 steps

### 1. Create the context

```tsx
// context/ThemeContext.tsx
import { createContext, useContext, useState, ReactNode } from 'react';

type Theme = 'light' | 'dark';

interface ThemeContextValue {
  theme: Theme;
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextValue | undefined>(undefined);
```

### 2. Create a Provider component

```tsx
export function ThemeProvider({ children }: { children: ReactNode }) {
  const [theme, setTheme] = useState<Theme>('dark');

  const toggleTheme = () =>
    setTheme(prev => (prev === 'dark' ? 'light' : 'dark'));

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}
```

### 3. Consume with a custom hook

```tsx
export function useTheme() {
  const ctx = useContext(ThemeContext);
  if (!ctx) throw new Error('useTheme must be used within ThemeProvider');
  return ctx;
}
```

### Usage in a component

```tsx
function Header() {
  const { theme, toggleTheme } = useTheme();

  return (
    <header className={theme}>
      <button onClick={toggleTheme}>Toggle Theme</button>
    </header>
  );
}
```

## When to use Context vs other solutions

| Scenario | Solution |
|----------|----------|
| Theme, locale, auth user | ✅ Context |
| UI state local to a subtree | ✅ Context |
| Complex global state with actions | Zustand / Redux |
| Server cache (API data) | React Query / SWR |

## Common Pitfalls

- **Every consumer re-renders when context value changes** — Split contexts by update frequency (e.g., separate `UserContext` and `UserDispatchContext`).
- **Object identity causing unnecessary re-renders** — Memoize the context value with `useMemo`.
- **Provider not wrapping the consumer** — The custom hook will throw; wrap your app (or the relevant subtree) with the Provider.
- **Overusing context for everything** — For purely local state, `useState` is simpler.

## Practice
```tsx
const UserContext = createContext(null);

function App() {
  const [user, setUser] = useState({ name: "Alex" });

  return (
    <UserContext.Provider value={user}>
      <Navbar />
      <button onClick={() => setUser({ name: "Sam" })}>Change User</button>
    </UserContext.Provider>
  );
}

function Navbar() {
  const user = useContext(UserContext);
  return <nav>Welcome, {user.name}!</nav>;
}
```

## Interview Questions
1. **How do you prevent unnecessary re-renders when using Context?**
   You can memoize the provider's value with `useMemo`, split the context into multiple contexts (one for state, one for dispatch), or use a state management library like Zustand.
2. **What is "Prop Drilling" and how does Context solve it?**
   Prop drilling is passing data through many layers of components that don't need it. Context allows a component to "teleport" data to any descendant without intermediate components being involved.

## Learn More
- [React Docs — useContext](https://react.dev/reference/react/useContext)
- [Passing Data Deeply with Context](https://react.dev/learn/passing-data-deeply-with-context)
