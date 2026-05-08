# `this` Keyword — Interview Questions

---

**1. What is `this` in JavaScript and why is its value not fixed?**

`this` refers to the object that owns the currently executing function. Its value is determined at **call time**, not at definition time (except for arrow functions). The same function can have a different `this` depending on how it is invoked — as a plain call, a method call, via `call`/`apply`/`bind`, or with `new`.

---

**2. Explain the four binding rules for `this`.**

| Rule | Example | `this` value |
|------|---------|--------------|
| Global | `foo()` | `window` (or `undefined` in strict mode) |
| Implicit | `obj.foo()` | `obj` |
| Explicit | `foo.call(obj)` | `obj` |
| New | `new Foo()` | Newly created instance |

Priority order: **new > explicit > implicit > global**

---

**3. What is the difference between `call`, `apply`, and `bind`?**

All three explicitly set `this`, but they differ in execution:
- `call(thisArg, arg1, arg2)` — invokes the function immediately, arguments listed individually.
- `apply(thisArg, [arg1, arg2])` — invokes immediately, arguments as a single array.
- `bind(thisArg, arg1)` — returns a *new* permanently-bound function, does not call it.

```javascript
function greet(greeting, mark) {
  console.log(`${greeting} ${this.name}${mark}`);
}
const user = { name: "Alice" };

greet.call(user, "Hello", "!");    // → "Hello Alice!"
greet.apply(user, ["Hi", "?"]);   // → "Hi Alice?"
const fn = greet.bind(user, "Hey");
fn(".");                           // → "Hey Alice."
```

---

**4. Why do arrow functions not have their own `this`?**

Arrow functions capture `this` from the lexical (surrounding) scope at the time they are *defined*. This solves the classic "lost this in a callback" problem:

```javascript
const obj = {
  value: 42,
  getValueLater() {
    setTimeout(() => {
      console.log(this.value); // 42 — arrow inherits from getValueLater's this
    }, 100);
  }
};
obj.getValueLater(); // → 42
```

With a regular function inside `setTimeout`, `this` would be `window` and `this.value` would be `undefined`.

---

**5. What is the output of this code and why?**

```javascript
const obj = {
  name: "Richa",
  greet: function() {
    setTimeout(function() {
      console.log(this.name);
    }, 100);
  }
};
obj.greet();
```

**Output**: `undefined`

The callback inside `setTimeout` is a regular function. When it runs, it is called as a plain function (not as `obj.greet`), so `this` defaults to `window`. `window.name` is `""` or `undefined`. Fix: use an arrow function inside `setTimeout`.

---

**6. What happens when you use an arrow function as an object method?**

It breaks — the arrow function inherits `this` from the enclosing scope (usually the module or `window`), not the object.

```javascript
const person = {
  name: "Bob",
  greet: () => {
    console.log(this.name); // → undefined (this = window, not person)
  }
};
person.greet(); // → undefined

// Fix: use regular method shorthand
const personFixed = {
  name: "Bob",
  greet() { console.log(this.name); } // → "Bob"
};
```

---

**7. How do you fix `this` binding in React class component event handlers?**

Three approaches:

```javascript
// Option 1 — bind in constructor
this.handleClick = this.handleClick.bind(this);

// Option 2 — class field arrow function (modern, recommended)
handleClick = () => { this.setState(...); };

// Option 3 — inline arrow in JSX
<button onClick={() => this.handleClick()}>Click</button>
```

Option 2 is the most common. Functional components with hooks avoid the issue entirely.

---

**8. What is "losing `this`" and how do you prevent it?**

Losing `this` occurs when a method is extracted from its object and called as a plain function:

```javascript
const wallet = { balance: 100, get() { return this.balance; } };
const fn = wallet.get;
fn(); // → undefined  (this is now window)
```

Prevention:
- `bind`: `const fn = wallet.get.bind(wallet);`
- Arrow wrapper: `() => wallet.get()`
- Keep the call on the object: `wallet.get()`

---

**9. What does `new` binding do step by step?**

When you call `new Foo(args)`, JavaScript:
1. Creates a fresh empty object `{}`.
2. Sets `this` inside the constructor to that object.
3. Executes the constructor body (properties attach to `this`).
4. Returns the object automatically.

```javascript
function Dog(name) {
  this.name = name;
  this.bark = () => `${this.name} says woof!`;
}
const d = new Dog("Rex");
console.log(d.bark()); // → "Rex says woof!"
```

---

**10. What is the `this` priority order when multiple rules could apply?**

From highest to lowest:
1. **`new` binding** — always wins
2. **Explicit** — `call`, `apply`, `bind`
3. **Implicit** — `obj.method()`
4. **Default** — plain `foo()` call

```javascript
function test() { console.log(this.x); }
const obj = { x: 10, test };

obj.test();                  // → 10  (implicit)
obj.test.call({ x: 99 });   // → 99  (explicit wins)
new obj.test();              // → undefined (new wins; fresh object has no x)
```
