- Category: JavaScript
- Difficulty: Intermediate
- Related: prototypes, this-keyword, es6-features

### ES6 Classes — Blueprints for Objects
A **class** is a reusable template (blueprint) for creating objects that share the same structure and behavior. Think of it as a cookie cutter — the cutter (class) stays the same, but every cookie (instance) is its own separate object with its own data.

**Analogy**
A car factory blueprint. The blueprint (class) describes every car: it has an engine, wheels, a brand name. Each actual car rolling off the line (instance) has its own color, mileage, and fuel level — but they all share the same structure.

---

### 1. Class Declaration & Constructor

**Theory**: The `class` keyword defines the blueprint. The `constructor` method runs automatically when you call `new ClassName()`. It sets up the initial state of the object using `this`.

**Working Flow**
![flow-chart](flow-chart.png)

**Example**
```javascript
class Car {
  constructor(brand, year) {
    this.brand = brand;
    this.year  = year;
    this.speed = 0;         // default value
  }

  describe() {
    return `${this.brand} (${this.year})`;
  }

  accelerate(amount) {
    this.speed += amount;
    return this.speed;
  }
}

const tesla = new Car("Tesla", 2023);
const bmw   = new Car("BMW",   2021);

console.log(tesla.describe());       // "Tesla (2023)"
console.log(bmw.describe());         // "BMW (2021)"
console.log(tesla.accelerate(60));   // 60
console.log(tesla.speed);            // 60
console.log(bmw.speed);              // 0  — separate instance!
```

**Output**
```
tesla.describe()     → "Tesla (2023)"
bmw.describe()       → "BMW (2021)"
tesla.accelerate(60) → 60
tesla.speed          → 60
bmw.speed            → 0
```

**Explanation**: Each `new Car(...)` call creates a completely independent object. Changing `tesla.speed` does not touch `bmw.speed`.

---

### 2. Static Methods — Class-Level Utilities

**Theory**: `static` methods belong to the **class itself**, not to instances. You call them on the class name, never on an object. Use them for utilities, factory helpers, or validation logic that does not need `this` data.

**Working Flow**
![flow-chart-2](flow-chart-2.png)

**Example**
```javascript
class MathHelper {
  static add(a, b)      { return a + b; }
  static multiply(a, b) { return a * b; }
  static isEven(n)      { return n % 2 === 0; }
  static clamp(n, min, max) {
    return Math.min(Math.max(n, min), max);
  }
}

console.log(MathHelper.add(3, 4));        // 7
console.log(MathHelper.multiply(3, 4));   // 12
console.log(MathHelper.isEven(7));        // false
console.log(MathHelper.clamp(150, 0, 100)); // 100

// Real-world: static factory method
class User {
  constructor(name, role) {
    this.name = name;
    this.role = role;
  }
  static createAdmin(name) {
    return new User(name, "admin");
  }
  static createGuest() {
    return new User("Guest", "guest");
  }
}

const admin = User.createAdmin("Alice");
const guest = User.createGuest();
console.log(admin);  // User { name: "Alice", role: "admin" }
console.log(guest);  // User { name: "Guest", role: "guest" }
```

**Output**
```
MathHelper.add(3,4)       → 7
MathHelper.multiply(3,4)  → 12
MathHelper.isEven(7)      → false
MathHelper.clamp(150,0,100) → 100
admin.role                → "admin"
guest.name                → "Guest"
```

**Explanation**: `User.createAdmin()` is a factory — it creates a `User` with preset values. This keeps object creation logic inside the class.

---

### 3. Getters & Setters

**Theory**: Getters (`get`) let you define a property that is computed on the fly when accessed. Setters (`set`) let you intercept and validate assignment. They look like properties from the outside but run functions internally.

**Working Flow**
![flow-chart-3](flow-chart-3.png)

**Example**
```javascript
class Person {
  constructor(firstName, lastName, age) {
    this.firstName = firstName;
    this.lastName  = lastName;
    this._age      = age;   // convention: _ = backing field
  }

  // getter — computed property
  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  }

  // setter — split "First Last" back into fields
  set fullName(val) {
    const parts = val.split(" ");
    this.firstName = parts[0];
    this.lastName  = parts[1] || "";
  }

  // getter with validation
  get age() {
    return this._age;
  }

  set age(val) {
    if (val < 0 || val > 150) throw new Error("Invalid age");
    this._age = val;
  }
}

const p = new Person("Jane", "Doe", 30);
console.log(p.fullName);   // "Jane Doe"
p.fullName = "Alice Smith";
console.log(p.firstName);  // "Alice"
console.log(p.lastName);   // "Smith"

p.age = 25;
console.log(p.age);        // 25
// p.age = -5;             // throws Error: Invalid age
```

**Output**
```
p.fullName (initial)   → "Jane Doe"
after set "Alice Smith"
  p.firstName          → "Alice"
  p.lastName           → "Smith"
p.age = 25             → 25
p.age = -5             → Error: Invalid age
```

**Explanation**: Getters/setters add a validation or transformation layer without changing how the property looks to the outside world.

---

### 4. Inheritance — extends & super

**Theory**: `extends` creates a child class that inherits all methods and properties from the parent. `super()` inside the child constructor calls the parent constructor to initialize inherited fields. Without calling `super()`, the child has no `this` and throws a ReferenceError.

**Working Flow**
![flow-chart-4](flow-chart-4.png)

**Example**
```javascript
class Animal {
  constructor(name, sound) {
    this.name  = name;
    this.sound = sound;
  }

  speak() {
    return `${this.name} says ${this.sound}`;
  }

  breathe() {
    return `${this.name} is breathing`;
  }
}

class Dog extends Animal {
  constructor(name) {
    super(name, "Woof");   // calls Animal constructor
    this.tricks = [];
  }

  learn(trick) {
    this.tricks.push(trick);
    return `${this.name} learned ${trick}`;
  }

  showTricks() {
    return `${this.name} knows: ${this.tricks.join(", ")}`;
  }
}

class Cat extends Animal {
  constructor(name, indoor) {
    super(name, "Meow");
    this.indoor = indoor;
  }

  info() {
    return `${this.name} is ${this.indoor ? "indoor" : "outdoor"}`;
  }
}

const dog = new Dog("Rex");
const cat = new Cat("Whiskers", true);

console.log(dog.speak());         // "Rex says Woof"
console.log(dog.breathe());       // "Rex is breathing"
console.log(dog.learn("sit"));    // "Rex learned sit"
console.log(dog.learn("shake")); // "Rex learned shake"
console.log(dog.showTricks());   // "Rex knows: sit, shake"
console.log(cat.speak());         // "Whiskers says Meow"
console.log(cat.info());          // "Whiskers is indoor"
```

**Output**
```
dog.speak()       → "Rex says Woof"
dog.breathe()     → "Rex is breathing"
dog.learn("sit")  → "Rex learned sit"
dog.showTricks()  → "Rex knows: sit, shake"
cat.speak()       → "Whiskers says Meow"
cat.info()        → "Whiskers is indoor"
```

**Explanation**: `Dog` and `Cat` both reuse `speak()` and `breathe()` from `Animal` without rewriting them. Each adds its own specific behavior.

---

### 5. Method Overriding

**Theory**: A child class can redefine a method that exists on the parent. JS will use the child's version (the most specific one). You can still call the parent's version inside the override using `super.methodName()`.

**Working Flow**
![flow-chart-5](flow-chart-5.png)

**Example**
```javascript
class Shape {
  constructor(color) {
    this.color = color;
  }
  area() {
    return 0;
  }
  describe() {
    return `A ${this.color} shape with area ${this.area().toFixed(2)}`;
  }
}

class Circle extends Shape {
  constructor(color, radius) {
    super(color);
    this.radius = radius;
  }
  area() {                             // overrides Shape.area
    return Math.PI * this.radius ** 2;
  }
}

class Rectangle extends Shape {
  constructor(color, width, height) {
    super(color);
    this.width  = width;
    this.height = height;
  }
  area() {                             // overrides Shape.area
    return this.width * this.height;
  }
  describe() {
    return super.describe() + ` [${this.width}x${this.height}]`; // calls parent
  }
}

const c = new Circle("red", 5);
const r = new Rectangle("blue", 4, 6);

console.log(c.area());        // 78.54
console.log(c.describe());    // "A red shape with area 78.54"
console.log(r.area());        // 24
console.log(r.describe());    // "A blue shape with area 24.00 [4x6]"
```

**Output**
```
c.area()       → 78.54
c.describe()   → "A red shape with area 78.54"
r.area()       → 24
r.describe()   → "A blue shape with area 24.00 [4x6]"
```

**Explanation**: `describe()` in `Shape` calls `this.area()` — because of polymorphism, it calls the *correct* overridden version automatically.

---

### 6. Private Fields (#) & instanceof

**Theory**: Fields prefixed with `#` are truly private — they cannot be read or written from outside the class at all (not even in subclasses). `instanceof` checks whether an object was created from a particular class or its parent.

**Working Flow**
![flow-chart-6](flow-chart-6.png)

**Example**
```javascript
class BankAccount {
  #balance = 0;       // private field
  #owner;

  constructor(owner, initialDeposit = 0) {
    this.#owner   = owner;
    this.#balance = initialDeposit;
  }

  deposit(amount) {
    if (amount <= 0) throw new Error("Amount must be positive");
    this.#balance += amount;
    return this;       // allow chaining
  }

  withdraw(amount) {
    if (amount > this.#balance) throw new Error("Insufficient funds");
    this.#balance -= amount;
    return this;
  }

  getBalance() {
    return this.#balance;
  }

  statement() {
    return `${this.#owner}'s balance: $${this.#balance}`;
  }
}

const acc = new BankAccount("Alice", 100);

acc.deposit(200).deposit(50).withdraw(30);  // chaining
console.log(acc.getBalance());   // 320
console.log(acc.statement());    // "Alice's balance: $320"

// console.log(acc.#balance);   // SyntaxError!

// instanceof
class Vehicle {}
class Car extends Vehicle {}
const myCar = new Car();

console.log(myCar instanceof Car);      // true
console.log(myCar instanceof Vehicle);  // true (parent)
console.log(myCar instanceof Array);    // false
```

**Output**
```
acc.getBalance()          → 320
acc.statement()           → "Alice's balance: $320"
acc.#balance              → SyntaxError
myCar instanceof Car      → true
myCar instanceof Vehicle  → true
myCar instanceof Array    → false
```

**Explanation**: Private fields enforce encapsulation at the language level. `instanceof` walks the prototype chain, so a child instance also passes the parent check.

---

### Real-World Example — Full BankAccount Class

```javascript
class BankAccount {
  #balance;
  #transactionHistory = [];

  constructor(owner, initialBalance = 0) {
    this.owner    = owner;
    this.#balance = initialBalance;
  }

  deposit(amount) {
    if (amount <= 0) throw new Error("Deposit must be positive");
    this.#balance += amount;
    this.#transactionHistory.push({ type: "deposit", amount, date: new Date().toLocaleDateString() });
    return this;
  }

  withdraw(amount) {
    if (amount <= 0)              throw new Error("Amount must be positive");
    if (amount > this.#balance)   throw new Error("Insufficient funds");
    this.#balance -= amount;
    this.#transactionHistory.push({ type: "withdraw", amount, date: new Date().toLocaleDateString() });
    return this;
  }

  get balance() { return this.#balance; }

  printStatement() {
    console.log(`\n=== ${this.owner}'s Statement ===`);
    this.#transactionHistory.forEach(t => {
      const sign = t.type === "deposit" ? "+" : "-";
      console.log(`  ${t.date}  ${sign}$${t.amount}  [${t.type}]`);
    });
    console.log(`  Current Balance: $${this.#balance}`);
  }
}

class SavingsAccount extends BankAccount {
  #interestRate;

  constructor(owner, balance, rate) {
    super(owner, balance);
    this.#interestRate = rate;
  }

  applyInterest() {
    const interest = this.balance * this.#interestRate;
    this.deposit(interest);
    console.log(`Interest of $${interest.toFixed(2)} applied`);
    return this;
  }
}

const savings = new SavingsAccount("Bob", 1000, 0.05);
savings.deposit(500).withdraw(200).applyInterest();
savings.printStatement();
```

**Output**
```
Interest of $65.00 applied

=== Bob's Statement ===
  [date]  +$500  [deposit]
  [date]  -$200  [withdraw]
  [date]  +$65   [deposit]
  Current Balance: $1365
```

---

[View Interview Questions](./interview.md)
