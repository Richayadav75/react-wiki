- Category: Fundamentals
- Difficulty: Beginner
- Related: variables

### What are Data Types?
Data types define the nature of data being stored. JavaScript has **Primitives** and **Reference** types.

---

### 1. Primitive Types
**Theory**: The most basic, unchangeable data types stored by value.

**Working Flow**
```text
[ Variable ] --> [ Direct Value ]
   (age)    -->      (25)
```

**Key Features**:
- **Number**: Integers and decimals (`10`, `3.14`).
- **String**: Text data wrapped in quotes (`"Hello"`).
- **Boolean**: Logical values (`true`, `false`).
- **Null**: Intentional empty value.

**Example**:
```javascript
let name = "Richa";
let age = 25;
let isStudent = true;
```

---

### 2. Reference Types
**Theory**: Complex structures that can store multiple values and are stored by reference in memory.

**Working Flow**
```text
[ Variable ] --> [ Memory Address (Pointer) ] --> [ Actual Data in Heap ]
   (user)   -->       (0x123abc)           -->   { name: "Richa" }
```

**Key Features**:
- **Objects**: Collections of key-value pairs.
- **Arrays**: Ordered lists of data.

**Example**:
```javascript
let user = { name: "Richa", city: "Delhi" };
let colors = ["Red", "Blue", "Green"];
```
