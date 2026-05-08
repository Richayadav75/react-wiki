# Classes & Objects — Interview Questions

---

**1. What is the difference between a class and an object in JavaScript?**

A **class** is a blueprint (template) that defines structure and behavior. An **object** is a concrete instance created from that blueprint using `new`. You can create many independent objects from one class, each with its own data.

```javascript
class Dog { constructor(name) { this.name = name; } }

const d1 = new Dog("Rex");
const d2 = new Dog("Buddy");
// d1 and d2 are separate objects — changing d1.name doesn't affect d2
```

---

**2. What does the `constructor` method do, and what happens if you don't define one?**

`constructor` runs automatically when `new` is called. It sets up initial properties on `this`. If you omit it, JavaScript inserts an empty default constructor silently. For a child class, the default constructor calls `super(...args)` automatically.

```javascript
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
}
const p = new Point(3, 4);
console.log(p.x, p.y); // 3 4
```

---

**3. What is the `static` keyword and when would you use it?**

`static` methods belong to the class itself, not to instances. Use them for utility functions, factory methods, or logic that does not depend on instance data.

```javascript
class Temperature {
  constructor(celsius) { this.celsius = celsius; }

  static fromFahrenheit(f) {
    return new Temperature((f - 32) * 5 / 9);
  }

  static isFreezing(celsius) { return celsius <= 0; }
}

const boiling = Temperature.fromFahrenheit(212);
console.log(boiling.celsius);            // 100
console.log(Temperature.isFreezing(0));  // true
```

---

**4. Explain `extends` and `super`. What happens if you forget `super()` in a child constructor?**

`extends` makes a class inherit from another. `super()` inside the child constructor calls the parent constructor to initialize inherited fields. Without `super()`, accessing `this` in the child constructor throws: `ReferenceError: Must call super constructor before accessing 'this'`.

```javascript
class Animal {
  constructor(name) { this.name = name; }
}
class Cat extends Animal {
  constructor(name, indoor) {
    super(name);          // must be FIRST
    this.indoor = indoor;
  }
}
const c = new Cat("Luna", true);
console.log(c.name, c.indoor); // "Luna" true
```

---

**5. What are private fields (`#`) and how do they differ from the `_` convention?**

Private fields (`#`) are enforced by the JavaScript engine — accessing them outside the class causes a `SyntaxError` at parse time. The `_name` underscore convention is purely a social agreement; nothing stops outside code from reading `obj._name`.

```javascript
class Wallet {
  #pin;                         // truly private
  _balance = 0;                 // convention only — still accessible

  constructor(pin) { this.#pin = pin; }

  verify(pin) { return this.#pin === pin; }
}

const w = new Wallet(1234);
console.log(w.verify(1234));  // true
console.log(w._balance);      // 0 — accessible (convention only)
// console.log(w.#pin);       // SyntaxError
```

---

**6. What is method overriding? How do you still call the parent method from the child?**

Method overriding is when a child class defines a method with the same name as the parent's. JS calls the child's version. To run the parent version from inside the child, use `super.methodName()`.

```javascript
class Logger {
  log(msg) { console.log(`[LOG] ${msg}`); }
}
class TimestampLogger extends Logger {
  log(msg) {
    super.log(msg);                        // calls parent
    console.log(`  at ${Date.now()}`);
  }
}
const tl = new TimestampLogger();
tl.log("Hello");
// [LOG] Hello
// at 1234567890
```

---

**7. What is the purpose of getters and setters? Give a practical example.**

Getters/setters let you intercept property access and assignment to add validation or compute derived values — while looking like regular properties to the caller.

```javascript
class Circle {
  constructor(radius) { this._radius = radius; }

  get radius()  { return this._radius; }
  set radius(r) {
    if (r < 0) throw new Error("Radius cannot be negative");
    this._radius = r;
  }
  get area()    { return (Math.PI * this._radius ** 2).toFixed(2); }
}

const c = new Circle(5);
console.log(c.area);    // "78.54"
c.radius = 10;
console.log(c.area);    // "314.16"
// c.radius = -1;       // Error: Radius cannot be negative
```

---

**8. How does `instanceof` work with inheritance?**

`instanceof` checks whether a constructor's `prototype` exists anywhere in the object's prototype chain. Because inheritance creates a chain, an instance of a child class also passes `instanceof` for the parent.

```javascript
class Vehicle {}
class Car extends Vehicle {}
class Boat extends Vehicle {}

const c = new Car();
console.log(c instanceof Car);      // true
console.log(c instanceof Vehicle);  // true  (parent)
console.log(c instanceof Boat);     // false
```

---

**9. What is the difference between instance methods and static methods?**

| | Instance Method | Static Method |
|---|---|---|
| Called on | `obj.method()` | `ClassName.method()` |
| Has `this` | Yes — refers to the instance | Yes — refers to the class |
| Use case | Work with instance data | Utilities, factories |

```javascript
class Counter {
  constructor() { this.count = 0; }
  increment() { this.count++; }        // instance
  static create() { return new Counter(); }  // static
}
const c = Counter.create();
c.increment();
console.log(c.count); // 1
```

---

**10. Can a class extend multiple classes? What is the workaround?**

No. JavaScript only supports **single inheritance** (`extends` one class). The workaround is **mixins** — functions that copy methods from multiple source objects onto a class prototype.

```javascript
const Serializable = (Base) => class extends Base {
  serialize()   { return JSON.stringify(this); }
  static parse(json) { return Object.assign(new this(), JSON.parse(json)); }
};

const Timestamped = (Base) => class extends Base {
  constructor(...args) { super(...args); this.createdAt = Date.now(); }
};

class User { constructor(name) { this.name = name; } }

class EnhancedUser extends Serializable(Timestamped(User)) {}

const u = new EnhancedUser("Alice");
console.log(u.serialize());   // {"name":"Alice","createdAt":...}
```
