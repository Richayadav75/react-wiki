- Category: JavaScript
- Difficulty: Advanced
- Related: Classes, Inheritance

# Prototypes

Prototypes are the mechanism by which JavaScript objects inherit features from one another.

## The Prototype Chain
Every object in JavaScript has a private property which holds a link to another object called its prototype. That prototype object has a prototype of its own, and so on until an object is reached with null as its prototype.

## Property Shadowing
If an object and its prototype both have a property with the same name, the property on the object is used first.

## Example
```javascript
const animal = {
  eat: function() {
    console.log("Eating...");
  }
};

const dog = Object.create(animal);
dog.bark = function() {
  console.log("Barking!");
};

dog.eat(); // Inherited: Eating...
dog.bark(); // Own: Barking!
```
