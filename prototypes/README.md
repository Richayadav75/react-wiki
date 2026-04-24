- Category: JavaScript
- Difficulty: Advanced
- Related: classes-objects

### What is a Prototype?
In JavaScript, almost everything is an object. Prototypes are the mechanism by which JavaScript objects inherit features from one another.

---

### 1. Prototypal Inheritance
**Theory**: Every object has a hidden, internal property called `[[Prototype]]`. When you try to access a method on an object, JS first looks on the object itself. If it doesn't find it, it looks up the "Prototype Chain".

**Working Flow**
```text
[ myObj ] --( finds nothing )--> [ myObj's Prototype ] --( finds toString() )--> [ Returns Value ]
```

**Key Features**:
- **Object.create()**: Can be used to manually set an object's prototype.
- **Memory Efficiency**: Methods placed on a prototype are shared among all instances, saving memory.

**Step-by-Step Example**:
```javascript
// Step 1: A simple object with a method
const animal = {
  eats: true,
  walk() {
    console.log("Animal walks");
  }
};

// Step 2: Create a new object that INHERITS from 'animal'
const rabbit = Object.create(animal);
rabbit.jumps = true;

// Step 3: Access properties
console.log(rabbit.jumps); // true (found directly on rabbit)
console.log(rabbit.eats);  // true (found on prototype: animal)
rabbit.walk();             // "Animal walks" (found on prototype: animal)
```

---

[View Interview Questions](./interview.md)
