- Category: State Management
- Difficulty: Intermediate
- Related: useReducer

### What is Context API?
The Context API is a way to pass data through the component tree without having to pass props down manually at every level ("prop drilling").

---

### 1. Global Data Sharing
**Theory**: It allows you to create a "Global State" that any component can subscribe to, no matter how deep it is in the tree.

**Working Flow**
```text
[ Provider (Parent) ] --( Broadcast Data )--> [ Consumer (Any Child) ]
```

**Key Features**:
- **No Prop Drilling**: Skips middle components that don't need the data.
- **Provider**: Wraps the components that need the data.
- **Consumer**: Hooks into the context to read values.

**Example (Theme Switching)**:
```tsx
// 1. Create Context
const ThemeContext = createContext('light');

// 2. Provide Value
<ThemeContext.Provider value="dark">
  <App />
</ThemeContext.Provider>

// 3. Use Value (in any child)
const theme = useContext(ThemeContext);
```

---

[View Interview Questions](./interview.md)
