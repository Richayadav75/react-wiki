- Category: JavaScript
- Difficulty: Intermediate
- Related: Prototypes

# Classes

Introduced in ES6, classes provide a much cleaner and more intuitive syntax for creating objects and dealing with inheritance, though they are primarily "syntactic sugar" over JavaScript's prototypal inheritance.

## Syntax

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }

  speak() {
    console.log(`${this.name} makes a noise.`);
  }
}

class Dog extends Animal {
  speak() {
    console.log(`${this.name} barks.`);
  }
}

const d = new Dog('Rex');
d.speak(); // Rex barks.
```

## Static Methods
Methods defined with `static` are called on the class itself, not on instances.

```javascript
class MathUtils {
  static square(x) {
    return x * x;
  }
}

console.log(MathUtils.square(4)); // 16
```

## Practice: Private Fields
JavaScript now supports private class fields using the `#` prefix.

```javascript
class BankAccount {
  #balance = 0;

  deposit(amount) {
    this.#balance += amount;
  }

  getBalance() {
    return this.#balance;
  }
}

const account = new BankAccount();
account.deposit(100);
// console.log(account.#balance); // ❌ Private field error
console.log(account.getBalance()); // 100
```

## Interview Questions
1. **Are JavaScript classes just like Java/C++ classes?**
   No, they are still based on prototypes. They don't introduce a new object-oriented inheritance model.
2. **What does the `super` keyword do?**
   `super` is used to call the constructor or methods of the parent class.
