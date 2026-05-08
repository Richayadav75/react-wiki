- Category: JavaScript
- Difficulty: Intermediate
- Related: closures, es6-features, functions

### The `this` Keyword — Who Is Speaking?
`this` is a special keyword that refers to the **object that owns the currently running function**. Think of it as a pronoun — just like "I" changes meaning depending on who says it, `this` changes meaning depending on *how* a function is called.

**Analogy**
A microphone at a press conference. Whoever holds the mic right now is "the speaker" — that is `this`. Pass the mic to someone else (`call`/`apply`/`bind`), and `this` changes. Arrow functions are like a speaker who keeps reading from a pre-written script — the author's voice never changes.

---

### 1. Global Binding — the default
**Theory**: When a function is called with no owner (plain function call), `this` defaults to the global object — `window` in the browser, `global` in Node.js. In **strict mode** it is `undefined` instead.

**Working Flow**
![flow-chart](flow-chart.png)

**Example**
```javascript
function showThis() {
  console.log(this);
}

showThis(); // → window (browser) or global (Node)

"use strict";
function strictThis() {
  console.log(this);
}
strictThis(); // → undefined
```

**Output**
```
showThis()    → Window { ... }   (browser)
strictThis()  → undefined        (strict mode)
```

**Explanation**: No object before the dot means no implicit owner, so JS falls back to the global object (or `undefined` in strict mode). This is the most common source of `this`-related bugs.

---

### 2. Implicit Binding — method on an object
**Theory**: When you call a function as a method of an object (`obj.method()`), `this` is automatically bound to that object. The rule: **look at what is to the left of the dot**.

**Working Flow**
![flow-chart-2](flow-chart-2.png)

**Example**
```javascript
const person = {
  name: "Alice",
  age: 28,
  greet() {
    console.log(`Hi, I am ${this.name} and I am ${this.age}`);
  },
  getAge: function() {
    return this.age;
  }
};

person.greet();      // → "Hi, I am Alice and I am 28"
person.getAge();     // → 28

// Nested objects — this = the IMMEDIATE owner
const company = {
  name: "TechCorp",
  ceo: {
    name: "Bob",
    introduce() {
      console.log(`I am ${this.name} from ${this.name}`);
      // this.name = "Bob" (ceo object, not company)
    }
  }
};
company.ceo.introduce(); // → "I am Bob from Bob"
```

**Output**
```
person.greet()           → "Hi, I am Alice and I am 28"
person.getAge()          → 28
company.ceo.introduce()  → "I am Bob from Bob"
```

**Explanation**: `this` always binds to the *closest* object before the dot — not the outermost object in the chain.

---

### 3. Explicit Binding — call, apply, bind
**Theory**: You can manually choose what `this` is using three methods. All three override implicit and global binding.

**Working Flow**
![flow-chart-3](flow-chart-3.png)

**Example**
```javascript
function introduce(greeting, punctuation) {
  console.log(`${greeting}, I am ${this.name}${punctuation}`);
}

const alice = { name: "Alice" };
const bob   = { name: "Bob"   };

// call — args listed individually, runs NOW
introduce.call(alice, "Hello", "!");   // → "Hello, I am Alice!"
introduce.call(bob,   "Hey",   ".");   // → "Hey, I am Bob."

// apply — args as an array, runs NOW
introduce.apply(alice, ["Hi", "?"]);   // → "Hi, I am Alice?"

// bind — returns a NEW function, runs LATER
const aliceIntro = introduce.bind(alice, "Welcome");
aliceIntro("...");  // → "Welcome, I am Alice..."

// bind for event handlers
const counter = {
  count: 0,
  increment() { this.count++; console.log(this.count); }
};
const inc = counter.increment.bind(counter);
setTimeout(inc, 1000); // → 1  (this is still counter, not window)
```

**Output**
```
introduce.call(alice,"Hello","!")  → "Hello, I am Alice!"
introduce.call(bob,"Hey",".")      → "Hey, I am Bob."
introduce.apply(alice,["Hi","?"])  → "Hi, I am Alice?"
aliceIntro("...")                  → "Welcome, I am Alice..."
setTimeout(inc, 1000)              → 1
```

**Explanation**: `call` and `apply` differ only in how arguments are passed. `bind` is unique — it does not call the function; it returns a permanently re-bound copy.

---

### 4. New Binding — constructor functions
**Theory**: When you call a function with `new`, JavaScript creates a brand-new empty object, sets `this` to that object, runs the function body, and returns the object automatically.

**Working Flow**
![flow-chart-4](flow-chart-4.png)

**Example**
```javascript
function Person(name, role) {
  this.name = name;
  this.role = role;
  this.greet = function() {
    return `${this.name} is a ${this.role}`;
  };
}

const alice = new Person("Alice", "Developer");
const bob   = new Person("Bob",   "Designer");

console.log(alice.greet()); // → "Alice is a Developer"
console.log(bob.greet());   // → "Bob is a Designer"
console.log(alice.name);    // → "Alice"

// Class syntax uses the same new binding under the hood
class Car {
  constructor(make, model) {
    this.make  = make;
    this.model = model;
  }
  describe() {
    return `${this.make} ${this.model}`;
  }
}

const car = new Car("Toyota", "Corolla");
console.log(car.describe()); // → "Toyota Corolla"
```

**Output**
```
alice.greet()  → "Alice is a Developer"
bob.greet()    → "Bob is a Designer"
alice.name     → "Alice"
car.describe() → "Toyota Corolla"
```

**Explanation**: `new` is the most unambiguous binding — `this` is always the fresh object being built. Classes use `new` binding automatically.

---

### 5. Arrow Functions — lexical `this`
**Theory**: Arrow functions do **not** have their own `this`. They inherit `this` from the enclosing scope at the time the arrow function is *defined* (not called). This is called **lexical binding**.

**Working Flow**
![flow-chart-5](flow-chart-5.png)

**Example**
```javascript
// Problem with regular function in a callback
const team = {
  name: "Alpha",
  members: ["Alice", "Bob", "Carol"],
  printMembers() {
    // 'this' here = team (implicit binding)
    this.members.forEach(function(member) {
      // 'this' here = window! (regular function, no owner)
      console.log(`${this.name}: ${member}`); // BROKEN
    });
  }
};

// Fix with arrow function
const teamFixed = {
  name: "Alpha",
  members: ["Alice", "Bob", "Carol"],
  printMembers() {
    // 'this' here = teamFixed
    this.members.forEach((member) => {
      // Arrow inherits 'this' from printMembers → teamFixed
      console.log(`${this.name}: ${member}`); // WORKS
    });
  }
};

teamFixed.printMembers();

// setTimeout example
const timer = {
  seconds: 0,
  start() {
    setInterval(() => {
      this.seconds++; // 'this' = timer (lexical)
      console.log(this.seconds);
    }, 1000);
  }
};
timer.start(); // logs 1, 2, 3 ...
```

**Output**
```
teamFixed.printMembers():
  → "Alpha: Alice"
  → "Alpha: Bob"
  → "Alpha: Carol"

timer.start():
  → 1 (after 1s)
  → 2 (after 2s)
  ...
```

**Explanation**: Arrow functions solve the "lost this" problem inside callbacks. Never use an arrow function as an object method itself though — it would inherit `this` from outside the object (usually `window`).

---

### 6. `this` in React Event Handlers
**Theory**: In React class components, event handler methods lose their `this` binding when passed as callbacks. Three common solutions exist: binding in constructor, class field arrow functions, and inline arrow functions.

**Working Flow**
![flow-chart-6](flow-chart-6.png)

**Example**
```javascript
// Problem
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
  }

  increment() {
    // this is undefined here when passed as onClick!
    this.setState({ count: this.state.count + 1 });
  }

  render() {
    return <button onClick={this.increment}>Click</button>; // BROKEN
  }
}

// Fix 1: Bind in constructor (most explicit)
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
    this.increment = this.increment.bind(this); // bind once
  }
  increment() { this.setState({ count: this.state.count + 1 }); }
  render() { return <button onClick={this.increment}>Click</button>; }
}

// Fix 2: Class field arrow function (modern, recommended)
class Counter extends React.Component {
  state = { count: 0 };
  increment = () => {   // arrow function = lexical this = component instance
    this.setState({ count: this.state.count + 1 });
  };
  render() { return <button onClick={this.increment}>Click</button>; }
}

// Fix 3: Inline arrow (creates new fn on each render — OK for simple cases)
class Counter extends React.Component {
  state = { count: 0 };
  increment() { this.setState({ count: this.state.count + 1 }); }
  render() {
    return <button onClick={() => this.increment()}>Click</button>;
  }
}
```

**Output**
```
Fix 1 (bind):         onClick → increment() runs, this = component ✓
Fix 2 (arrow field):  onClick → increment() runs, this = component ✓
Fix 3 (inline arrow): onClick → increment() runs, this = component ✓
```

**Explanation**: React functional components with hooks avoid this problem entirely — `this` is not used at all in function components.

---

### 7. Common Pitfalls — losing `this`
**Theory**: `this` is lost when a method is extracted from its object (assigned to a variable or passed as a callback). The function is no longer called as a method, so implicit binding is gone.

**Working Flow**
![flow-chart-7](flow-chart-7.png)

**Example**
```javascript
const wallet = {
  balance: 500,
  getBalance() { return this.balance; }
};

// Works — implicit binding
console.log(wallet.getBalance());  // → 500

// BROKEN — method extracted, this lost
const getB = wallet.getBalance;
console.log(getB()); // → undefined (this.balance is window.balance)

// Fix with bind
const safeGetB = wallet.getBalance.bind(wallet);
console.log(safeGetB()); // → 500

// BROKEN — passed as callback
setTimeout(wallet.getBalance, 0); // → undefined

// Fix with arrow wrapper
setTimeout(() => wallet.getBalance(), 0); // → 500
```

**Output**
```
wallet.getBalance()      → 500       ✓
getB()                   → undefined ✗  (this lost)
safeGetB()               → 500       ✓  (bind fixes it)
setTimeout(getBalance)   → undefined ✗  (this lost)
setTimeout(arrow wrapper)→ 500       ✓  (arrow captures wallet)
```

---

### Real-World Practical Example
**Building a user session manager**
```javascript
class SessionManager {
  constructor() {
    this.user     = null;
    this.loggedIn = false;
    // Bind so these can be passed freely as callbacks
    this.login  = this.login.bind(this);
    this.logout = this.logout.bind(this);
  }

  login(userData) {
    this.user     = userData;
    this.loggedIn = true;
    console.log(`Logged in: ${this.user.name}`);
  }

  logout() {
    console.log(`Goodbye, ${this.user.name}`);
    this.user     = null;
    this.loggedIn = false;
  }

  status() {
    return this.loggedIn ? `${this.user.name} is online` : "No active session";
  }
}

const session = new SessionManager();
session.login({ name: "Alice", role: "admin" });
console.log(session.status());
session.logout();
console.log(session.status());
```

**Output**
```
Logged in: Alice
Alice is online
Goodbye, Alice
No active session
```

---

[View Interview Questions](./interview.md)
