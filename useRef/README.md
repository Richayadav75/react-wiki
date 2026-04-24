- Category: Hooks
- Difficulty: Intermediate
- Related: useState

### What is useRef?
`useRef` is used to persist values between renders without triggering a re-render. It is also the primary way to access DOM elements directly.

---

### 1. Persistent Storage
**Theory**: Unlike state, changing a ref's value does NOT make the component re-render. It's like a private "box" that stays with the component.

**Key Features**:
- **Mutable**: You can change `.current` directly.
- **DOM Access**: Used to focus inputs or measure elements.

**Example (Input Focus)**:
```tsx
const inputRef = useRef(null);

const focusInput = () => {
  inputRef.current.focus(); // Direct DOM manipulation
};

return <input ref={inputRef} />;
```

---

[View Interview Questions](./interview.md)
