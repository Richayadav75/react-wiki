- Category: JavaScript
- Difficulty: Intermediate
- Related: prototypes

### What are Classes & Objects?
Object-Oriented Programming (OOP) in JavaScript allows you to structure your code using "Classes" as blueprints to create individual "Objects".

---

### 1. The Class Blueprint
**Theory**: A class defines the properties (data) and methods (behavior) that its objects will have.

**Working Flow**
```text
[ Blueprint (Class) ] ---> ( new ) ---> [ Instance (Object) ]
      (Car)                               (My Honda)
```

**Key Features**:
- **constructor**: A special method that runs automatically when a new object is created.
- **this**: Refers to the specific object currently being created or used.

**Step-by-Step Example**:
```javascript
// Step 1: Define the blueprint
class User {
  // Step 2: The constructor sets up the initial data
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  // Step 3: Add methods (behavior)
  greet() {
    console.log(`Hi, I am ${this.name}`);
  }
}

// Step 4: Create actual objects using the 'new' keyword
const user1 = new User("Richa", 25);
user1.greet(); // "Hi, I am Richa"
```

---

[View Interview Questions](./interview.md)
