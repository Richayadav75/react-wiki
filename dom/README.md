- Category: DOM Manipulation
- Difficulty: Beginner
- Related: events

### What is the DOM?
The DOM (Document Object Model) is an API for HTML documents. It represents the page so that programs can change the document structure, style, and content.

---

### 1. Selecting Elements
**Theory**: Before you can change an element, you must find it in the DOM tree.

**Working Flow**
```text
[ HTML Document ] ---> ( document.querySelector ) ---> [ HTML Element ]
```

**Key Methods**:
- `getElementById()`: Selects a single element by its ID.
- `querySelector()`: Selects the first element that matches a CSS selector.
- `querySelectorAll()`: Selects ALL elements that match a CSS selector (returns a NodeList).

**Step-by-Step Example**:
```javascript
// Step 1: Select an element by its ID
const title = document.getElementById("main-title");

// Step 2: Select the first button with the class 'btn-primary'
const button = document.querySelector(".btn-primary");

// Step 3: Select all list items inside a specific ul
const items = document.querySelectorAll("ul#nav li");
```

---

### 2. Manipulating Elements
**Theory**: Once an element is selected, you can change its text, HTML content, or CSS styles.

**Working Flow**
```text
[ <p>Old</p> ] --( textContent = "New" )--> [ <p>New</p> ]
```

**Key Properties**:
- `textContent`: Changes the text inside an element (safe from XSS).
- `innerHTML`: Changes the HTML inside an element.
- `style`: Accesses the inline CSS styles.

**Step-by-Step Example**:
```javascript
const box = document.querySelector(".box");

// Step 1: Change the text
box.textContent = "Hello World!";

// Step 2: Change the HTML structure inside it
box.innerHTML = "<strong>Bold Text!</strong>";

// Step 3: Change its CSS styling dynamically
box.style.backgroundColor = "blue";
box.style.fontSize = "24px";
```
