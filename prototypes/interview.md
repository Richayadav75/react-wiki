# Prototypes — Interview Questions

---

**1. What is the prototype chain? How does JavaScript look up a property?**

Every object has an internal `[[Prototype]]` link to another object. When you access a property, JS first checks the object itself. If not found, it checks the prototype, then the prototype's prototype, continuing until it reaches `null`. This sequence is the **prototype chain**.

```javascript
const grandparent = { alive: true };
const parent      = Object.create(grandparent);
parent.smart = true;
const child       = Object.create(parent);

console.log(child.smart); // true  (found on parent)
console.log(child.alive); // true  (found on grandparent)
console.log(child.unknown); // undefined (chain ends at null)
```

---

**2. What is the difference between `__proto__` and `prototype`?**

- `prototype` is a property on **constructor functions/classes**. It is the object that becomes `__proto__` of every new instance.
- `__proto__` is a property on **instances** that points to their actual prototype object.

```javascript
function Dog(name) { this.name = name; }
Dog.prototype.bark = function() { return "Woof!"; };

const d = new Dog("Rex");

console.log(d.__proto__ === Dog.prototype);          // true
console.log(Object.getPrototypeOf(d) === Dog.prototype); // true (preferred)
console.log(d.prototype);                            // undefined (only on functions)
```

---

**3. What does `Object.create(null)` do and why would you use it?**

`Object.create(null)` creates an object with **no prototype at all** — not even `Object.prototype`. It has no `toString`, `hasOwnProperty`, or any inherited method. Use it for pure dictionary/hash-map objects where you don't want prototype pollution risks.

```javascript
const dict = Object.create(null);
dict["key"] = "value";

console.log(dict.toString);      // undefined — no inherited methods
console.log(dict.hasOwnProperty); // undefined
console.log("key" in dict);      // true — in still works
```

---

**4. What is `hasOwnProperty` and when should you use it?**

`hasOwnProperty(key)` returns `true` only if the property lives directly on the object — not inherited from a prototype. Use it inside `for...in` loops to avoid processing inherited properties.

```javascript
const base  = { inherited: true };
const child = Object.create(base);
child.own   = "mine";

for (const key in child) {
  if (child.hasOwnProperty(key)) {
    console.log("Own:", key);       // Own: own
  }
}

// Modern alternative
console.log(Object.hasOwn(child, "own"));       // true
console.log(Object.hasOwn(child, "inherited")); // false
```

---

**5. How does ES6 `class` relate to prototypes?**

`class` is **syntactic sugar** over the prototype system. Under the hood, `class Foo extends Bar` sets up the same prototype chain that was previously set up manually with `Object.create` and `call`.

```javascript
// ES6 class
class Animal { speak() { return "..."; } }
class Dog extends Animal { bark() { return "Woof"; } }

// What JS actually does internally:
// Dog.prototype = Object.create(Animal.prototype)
// Dog.prototype.constructor = Dog

const d = new Dog();
console.log(Object.getPrototypeOf(d) === Dog.prototype);      // true
console.log(Object.getPrototypeOf(Dog.prototype) === Animal.prototype); // true
```

---

**6. Why is adding methods to a prototype more memory-efficient than adding them inside the constructor?**

Methods defined inside the constructor create a **new function object for every instance**. Methods on the prototype are created **once** and shared by all instances via the chain lookup.

```javascript
// Wasteful — new greet() for every instance
function UserBad(name) {
  this.name  = name;
  this.greet = function() { return `Hi, ${this.name}`; }; // new fn each time
}

// Efficient — one greet shared via prototype
function UserGood(name) { this.name = name; }
UserGood.prototype.greet = function() { return `Hi, ${this.name}`; };

const u1 = new UserBad("A");
const u2 = new UserBad("B");
console.log(u1.greet === u2.greet); // false — different functions!

const g1 = new UserGood("A");
const g2 = new UserGood("B");
console.log(g1.greet === g2.greet); // true  — same function!
```

---

**7. What is prototype pollution and how do you prevent it?**

Prototype pollution is when an attacker sets a property on `Object.prototype` (e.g., via `__proto__` in JSON), causing every object in the app to inherit that property. Prevent it by using `Object.keys()` instead of `for...in`, checking for dangerous keys, or using `Object.create(null)`.

```javascript
// Vulnerable
const payload = JSON.parse('{"__proto__": {"isAdmin": true}}');
Object.assign({}, payload);
console.log({}.isAdmin); // true — polluted!

// Safe: check keys or use Object.create(null)
function safeMerge(target, src) {
  for (const key of Object.keys(src)) {
    if (["__proto__","constructor","prototype"].includes(key)) continue;
    target[key] = src[key];
  }
  return target;
}
```

---

**8. How does the prototype chain differ for arrays, functions, and plain objects?**

Each built-in type has its own `.prototype` object in the chain:

```text
plain object:  obj → Object.prototype → null
array:         arr → Array.prototype → Object.prototype → null
function:      fn  → Function.prototype → Object.prototype → null
```

```javascript
const arr = [1, 2, 3];
console.log(arr.hasOwnProperty("map"));          // false — on Array.prototype
console.log(Array.prototype.hasOwnProperty("map")); // true

function foo() {}
console.log(foo.hasOwnProperty("call"));         // false — on Function.prototype
console.log(Function.prototype.hasOwnProperty("call")); // true
```

---

**9. What is `Object.getPrototypeOf` and why prefer it over `__proto__`?**

`Object.getPrototypeOf(obj)` is the **official, standardized** way to read the prototype. `__proto__` is a legacy accessor that was standardized only for compatibility — it is not recommended in modern code because it can be overridden or cause issues in some environments.

```javascript
class A {}
class B extends A {}
const b = new B();

console.log(Object.getPrototypeOf(b) === B.prototype); // true
console.log(Object.getPrototypeOf(B.prototype) === A.prototype); // true
```

---

**10. What happens when you assign a property that exists on the prototype — does it modify the prototype?**

No. Assigning to an instance creates an **own property** on that instance, which **shadows** (hides) the prototype property. The prototype is not changed.

```javascript
const proto = { color: "red" };
const obj   = Object.create(proto);

obj.color = "blue";              // creates own property, does NOT modify proto
console.log(obj.color);          // "blue" (own property wins)
console.log(proto.color);        // "red"  (prototype unchanged)
console.log(obj.hasOwnProperty("color")); // true — it's own now
```
