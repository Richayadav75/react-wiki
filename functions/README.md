- category: Basic Concepts
- track: Fundamentals
- difficulty: Beginner
- related: variables, data-types


### Function declaration
**Theory**: The classic way to define a function. Declarations are hoisted — you can call them before they appear in the file.

**Example**:
```javascript
// Declare a function
function greet(name) {
  return "Hello, " + name + "!";
}

// Call it
console.log(greet("Alice"));   // → Hello, Alice!
console.log(greet("Bob"));     // → Hello, Bob!

// Function with multiple parameters
function add(a, b) {
  return a + b;
}
console.log(add(3, 7));        // → 10

// Without return → undefined
function sayHi() {
  console.log("Hi!");
}
let result = sayHi();
console.log(result);           // → undefined

```
**Output**:
```
greet("Alice") → "Hello, 
Alice!" 
greet("Bob") → "Hello, 
Bob!" 
add(3, 7) → 10 
sayHi() 
result → undefined
```

**Explanation**:
```
function greet(name) {
function keyword, then name, then parameters in (). name is a placeholder — it gets the value you pass when calling.
return "Hello, " + name + "!";
return sends a value back to the caller. Execution stops here.
greet("Alice")
Calling the function — 'Alice' replaces name inside the function.
let result = sayHi();
A function without return implicitly returns undefined.
```
-------------------------------------------

### Function expression & default params
**Theory**: A function stored in a variable is a function expression. Not hoisted. Default parameters handle missing arguments gracefully.

**Example**:
```javascript
// Function expression
const square = function(x) {
  return x * x;
};
console.log(square(4));    // → 16

// Arrow function (shorter syntax)
const cube = (x) => x * x * x;
console.log(cube(3));      // → 27

// Default parameters
function greet(name = "stranger", greeting = "Hello") {
  return greeting + ", " + name + "!";
}
console.log(greet());               // → Hello, stranger!
console.log(greet("Ana"));          // → Hello, Ana!
console.log(greet("Ana", "Hi"));    // → Hi, Ana!

```
**Explanation**:
const square = function(x) {
Function stored in a variable. Can't be called before this line (not hoisted).
const cube = (x) => x * x * x;
Arrow function with implicit return. No braces = single expression returned.
name = "stranger"
Default value used when argument is missing or undefined.
greet("Ana")
Only first arg given — greeting uses its default 'Hello'.
```

**Output**:
square(4) → 16 
cube(3) → 27 
greet() → "Hello, stranger!" 
greet("Ana") → "Hello, Ana!" 
greet("Ana","Hi") → "Hi, Ana!"
```
Click the topic tabs at the top to switch between the 5 foundation topics
Click any concept card to expand it — you'll see the code, then a line-by-line breakdown of what each part does
Hit "Run example" to execute the code live and see real output
---