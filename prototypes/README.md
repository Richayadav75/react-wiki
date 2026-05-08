- Category: JavaScript
- Difficulty: Advanced
- Related: classes-objects, this-keyword, functions

### Prototypes — JavaScript's Inheritance Engine
Every object in JavaScript has a hidden internal link to another object called its **prototype**. When you access a property that doesn't exist on the object, JavaScript automatically walks up this chain of prototypes looking for it. This lookup chain is the foundation of all inheritance in JS — even ES6 classes use it under the hood.

**Analogy**
A family tree. You ask a child for a skill. They don't know it — they ask their parent. Parent doesn't know — asks grandparent. Grandparent knows! The answer travels back down. If nobody knows (reaches `null`), the answer is `undefined`.

---

### 1. The Prototype Chain

**Theory**: Every object is secretly linked to a prototype object via an internal slot called `[[Prototype]]`. JavaScript follows this chain until it finds the property or hits `null` (the end of all chains).

**Working Flow**
![flow-chart](flow-chart.png)

**Example**
```javascript
const animal = { breathes: true };
const dog    = Object.create(animal); // dog's prototype = animal
dog.sound = "Woof";

console.log(dog.sound);    // "Woof"   — own property
console.log(dog.breathes); // true     — inherited from animal
console.log(dog.unknown);  // undefined — not found anywhere

// Checking the chain
console.log(Object.getPrototypeOf(dog) === animal); // true
console.log(Object.getPrototypeOf(animal) === Object.prototype); // true
console.log(Object.getPrototypeOf(Object.prototype)); // null
```

**Output**
```
dog.sound          → "Woof"
dog.breathes       → true   (from animal)
dog.unknown        → undefined
proto of dog       → animal (true)
proto of animal    → Object.prototype (true)
proto of Object.prototype → null
```

**Explanation**: `dog` does not have `breathes` itself, so JS walks to `dog`'s prototype (`animal`), finds it there, and returns it.

---

### 2. `__proto__` vs `.prototype`

**Theory**: These two look similar but are completely different things used in different contexts.

**Working Flow**
![flow-chart-2](flow-chart-2.png)

**Example**
```javascript
function Animal(name) {
  this.name = name;
}
Animal.prototype.breathe = function() {
  return `${this.name} is breathing`;
};

const cat = new Animal("Luna");

// __proto__ (instance link — legacy)
console.log(cat.__proto__ === Animal.prototype); // true

// Object.getPrototypeOf (modern, preferred)
console.log(Object.getPrototypeOf(cat) === Animal.prototype); // true

// .prototype (only on functions/classes)
console.log(Animal.prototype.constructor === Animal); // true

// cat does NOT have .prototype property
console.log(cat.prototype); // undefined
```

**Output**
```
cat.__proto__ === Animal.prototype        → true
getPrototypeOf(cat) === Animal.prototype  → true
Animal.prototype.constructor === Animal  → true
cat.prototype                            → undefined
```

| | `prototype` | `__proto__` |
|---|---|---|
| Lives on | Constructor functions & classes | Object instances |
| Purpose | Template for new instances | Actual link to prototype |
| Access | `Fn.prototype` | `obj.__proto__` or `Object.getPrototypeOf(obj)` |

---

### 3. Prototype-Based Inheritance (Pre-ES6)

**Theory**: Before classes existed, developers manually set up the prototype chain using constructor functions and `Object.create`. This is exactly what ES6 `class`/`extends` compiles down to.

**Working Flow**
![flow-chart-3](flow-chart-3.png)

**Example**
```javascript
// Parent constructor
function Vehicle(make, speed) {
  this.make  = make;
  this.speed = speed;
}
Vehicle.prototype.describe = function() {
  return `${this.make} going ${this.speed}km/h`;
};

// Child constructor
function Car(make, speed, doors) {
  Vehicle.call(this, make, speed);  // call parent
  this.doors = doors;
}

// Set up prototype chain
Car.prototype = Object.create(Vehicle.prototype);
Car.prototype.constructor = Car;

// Add child-only method
Car.prototype.honk = function() {
  return `${this.make}: beep beep!`;
};

const c = new Car("Toyota", 120, 4);
console.log(c.describe()); // "Toyota going 120km/h"  (inherited)
console.log(c.honk());     // "Toyota: beep beep!"
console.log(c instanceof Car);     // true
console.log(c instanceof Vehicle); // true
```

**Output**
```
c.describe()        → "Toyota going 120km/h"
c.honk()            → "Toyota: beep beep!"
c instanceof Car    → true
c instanceof Vehicle→ true
```

**Explanation**: This pattern is exactly what `class Car extends Vehicle` does behind the scenes.

---

### 4. hasOwnProperty — Own vs Inherited

**Theory**: `hasOwnProperty(key)` returns `true` only if the property lives directly on the object, not on a prototype. This is essential when iterating objects to avoid picking up inherited properties.

**Working Flow**
![flow-chart-4](flow-chart-4.png)

**Example**
```javascript
const parent = { inherited: true };
const child  = Object.create(parent);
child.own = "mine";

console.log(child.hasOwnProperty("own"));        // true
console.log(child.hasOwnProperty("inherited"));  // false
console.log("inherited" in child);               // true (in = checks chain)

// Common use: safe for..in loop
for (const key in child) {
  if (child.hasOwnProperty(key)) {
    console.log("Own:", key);          // prints "own"
  } else {
    console.log("Inherited:", key);    // prints "inherited"
  }
}

// Modern alternative
console.log(Object.hasOwn(child, "own"));        // true (ES2022)
console.log(Object.keys(child));                 // ["own"]  — own only
```

**Output**
```
hasOwnProperty("own")       → true
hasOwnProperty("inherited") → false
"inherited" in child        → true
for..in loop:
  Own: own
  Inherited: inherited
Object.keys(child)          → ["own"]
```

---

### 5. How Array & String Methods Come from Prototypes

**Theory**: When you call `[1,2,3].map(...)`, the array `[1,2,3]` doesn't own `map`. JS walks the prototype chain to `Array.prototype`, where `map` lives — shared by every single array in existence. This is why prototype-based method sharing is memory-efficient.

**Working Flow**
![flow-chart-5](flow-chart-5.png)

**Example**
```javascript
const arr = [1, 2, 3];

// map does NOT live on arr
console.log(arr.hasOwnProperty("map"));         // false
// map lives on Array.prototype
console.log(Array.prototype.hasOwnProperty("map")); // true

// Same chain for strings
const str = "hello";
console.log(typeof str.toUpperCase);            // "function"
// toUpperCase lives on String.prototype
console.log(String.prototype.hasOwnProperty("toUpperCase")); // true

// You can (but shouldn't in prod) add to Array.prototype
Array.prototype.sum = function() {
  return this.reduce((a, b) => a + b, 0);
};
console.log([1, 2, 3].sum()); // 6

// But this is dangerous — see next section
delete Array.prototype.sum;  // clean up
```

**Output**
```
arr.hasOwnProperty("map")         → false
Array.prototype.hasOwnProperty("map") → true
typeof str.toUpperCase            → "function"
[1,2,3].sum()                     → 6
```

---

### 6. Prototype Pollution — The Danger

**Theory**: Prototype pollution happens when untrusted input is used to write properties onto `Object.prototype` (the root of all objects). Because every object inherits from `Object.prototype`, a polluted property appears on every object in the application — a serious security vulnerability.

**Working Flow**
![flow-chart-6](flow-chart-6.png)

**Example**
```javascript
// Dangerous merge function (simplified real-world example)
function unsafeMerge(target, source) {
  for (const key in source) {
    target[key] = source[key];    // no protection
  }
}

const userInput = JSON.parse('{"__proto__": {"hacked": true}}');
unsafeMerge({}, userInput);

console.log({}.hacked);           // true  — POLLUTED!

// Safe fix: use Object.hasOwn check
function safeMerge(target, source) {
  for (const key of Object.keys(source)) {  // only own keys
    if (key === "__proto__" || key === "constructor" || key === "prototype") continue;
    target[key] = source[key];
  }
  return target;
}

// Even safer: Object.create(null) — no prototype at all
const safe = Object.create(null);
console.log(safe.toString); // undefined — no inherited methods
```

**Output**
```
After unsafeMerge:
  {}.hacked            → true   (prototype polluted!)

safe = Object.create(null):
  safe.toString        → undefined (no prototype)
```

**Explanation**: Always use `Object.keys()` (not `for...in`) or validate keys when merging untrusted data. Libraries like lodash fixed this in v4.17.21.

---

### Real-World Pattern — Prototype-Based Mixin

```javascript
// Mixin: reusable behavior without inheritance
const Serializable = {
  serialize()   { return JSON.stringify(this); },
  toLog()       { return `[${this.constructor.name}] ${this.serialize()}`; }
};

const Validatable = {
  isValid() { return Object.keys(this).every(k => this[k] !== null); }
};

class Product {
  constructor(name, price) {
    this.name  = name;
    this.price = price;
  }
}

// Copy mixin methods onto prototype
Object.assign(Product.prototype, Serializable, Validatable);

const p = new Product("Phone", 799);
console.log(p.serialize());  // '{"name":"Phone","price":799}'
console.log(p.toLog());      // '[Product] {"name":"Phone","price":799}'
console.log(p.isValid());    // true
```

**Output**
```
p.serialize()  → '{"name":"Phone","price":799}'
p.toLog()      → '[Product] {"name":"Phone","price":799}'
p.isValid()    → true
```

---

[View Interview Questions](./interview.md)
