- Category: Core Concepts of JavaScript
- Difficulty: Intermediate
- Related: Call, Apply, Bind

# The "this" Keyword

In JavaScript, the `this` keyword refers to the object it belongs to. Its value is determined by how a function is called (runtime binding).

## Binding Rules
1. **Default Binding**: In a standalone function call, `this` refers to the global object (`window` in browsers, `global` in Node). In strict mode, it is `undefined`.
2. **Implicit Binding**: When a function is called as a method of an object, `this` refers to that object.
3. **Explicit Binding**: Using `.call()`, `.apply()`, or `.bind()` to explicitly set `this`.
4. **New Binding**: When a function is called with the `new` keyword, `this` refers to the newly created object.
5. **Arrow Functions**: Arrow functions do not have their own `this`. They inherit it from the lexical scope.

## Practice
```javascript
const obj = {
  name: "Wiki",
  greet: function() {
    console.log("Hello, " + this.name);
  },
  arrowGreet: () => {
    console.log("Hello, " + this.name);
  }
};

obj.greet(); // Hello, Wiki
obj.arrowGreet(); // Hello, undefined
```

## Interview Questions
1. **How does 'this' work in arrow functions?**
   Arrow functions inherit 'this' from their parent lexical scope; they don't have their own.
2. **What is the difference between call and apply?**
   Both invoke a function with a specific 'this', but `call` takes arguments individually while `apply` takes them as an array.
