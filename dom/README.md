- Category: JavaScript
- Difficulty: Beginner
- Related: events, bom, event-delegation

### DOM — The Document Object Model

The **DOM** is JavaScript's live representation of the HTML page. When a browser loads HTML, it parses it into a tree of objects (nodes) that JavaScript can read and modify. Every change you make to the DOM instantly updates what the user sees.

**Analogy**
The DOM is like a family tree for your webpage. `document` is the great-grandparent. `<html>` is the grandparent. `<body>` is the parent. Each `<div>`, `<p>`, and `<button>` is a child. You can walk this tree, find members by name, add new family members, or remove old ones — and the displayed page updates immediately.

---

### 1. The DOM Tree Structure

**Theory**: HTML becomes a tree of nodes. Every element, text node, and attribute is a node in the tree. Understanding the tree is the foundation for all DOM manipulation.

**Working Flow**
![flow-chart](flow-chart.png)

**Example**
```javascript
// Navigate the tree
document.documentElement   // → <html> element
document.head              // → <head> element
document.body              // → <body> element

document.body.children     // → HTMLCollection of direct children
document.body.firstChild   // → first child node (could be text node)
document.body.firstElementChild  // → first child ELEMENT
```

**Output**
```
document.documentElement → <html lang="en">
document.body.children   → HTMLCollection(3) [h1, ul, button]
```

---

### 2. Selecting Elements

**Theory**: Before you can change an element, you must find it. There are several selection methods — each with a different use case. `querySelector` and `querySelectorAll` are the most powerful because they accept any CSS selector.

**Working Flow**
![flow-chart-2](flow-chart-2.png)

**Example**
```javascript
// By ID — fastest, returns element or null
const btn = document.getElementById("btn");

// By CSS selector — accepts any CSS selector
const title = document.querySelector("h1");
const firstItem = document.querySelector(".item");
const input = document.querySelector("form input[type='email']");

// All matching elements → NodeList (static snapshot)
const items = document.querySelectorAll(".item");
items.forEach(item => console.log(item.textContent));

// Convert NodeList to Array for full array methods
const itemArray = Array.from(items);
const texts = itemArray.map(el => el.textContent);
```

**Output**
```
querySelector("h1")     → <h1>Hello</h1>
querySelector(".item")  → <li class="item">Apple</li>
querySelectorAll count  → 2 (NodeList)
forEach output          → Apple
                        → Banana
texts (map)             → ["Apple", "Banana"]
```

**Explanation**
- `getElementById` is fastest but only works for IDs.
- `querySelector` accepts any valid CSS selector — incredibly flexible.
- `querySelectorAll` returns a **static** NodeList — it won't update if elements are added later. Use `Array.from()` to convert to Array for `map/filter/reduce`.

---

### 3. Creating and Inserting Elements

**Theory**: Instead of injecting raw HTML strings, use the DOM API to build elements safely. This approach avoids XSS risks and gives you full control over the created nodes.

**Working Flow**
![flow-chart-3](flow-chart-3.png)

**Example**
```javascript
// 1. Create a new list item
const li = document.createElement("li");
li.textContent = "Mango";
li.classList.add("item", "new");
li.setAttribute("data-fruit", "mango");

// 2. Find the parent and add the item
const list = document.querySelector("#list");
list.appendChild(li);   // adds at end

// 3. Prepend — add at beginning
const firstLi = document.createElement("li");
firstLi.textContent = "Avocado";
list.prepend(firstLi);

// 4. Remove an item
const appleItem = document.querySelector(".item");
appleItem.remove(); // removes Apple (first .item)

// 5. Replace an element
const newTitle = document.createElement("h2");
newTitle.textContent = "Updated Title";
const oldTitle = document.querySelector("h1");
oldTitle.replaceWith(newTitle);
```

**Output** (DOM state after all operations)
```
<ul id="list">
  <li class="item new" data-fruit="avocado">Avocado</li>
  <li class="item">Banana</li>
  <li class="item new" data-fruit="mango">Mango</li>
</ul>
<h2>Updated Title</h2>
```

---

### 4. innerHTML vs textContent — Security Critical

**Theory**: Both set element content, but they behave very differently. `innerHTML` parses and renders HTML. `textContent` treats everything as plain text. Using `innerHTML` with user-provided data is a serious XSS (cross-site scripting) vulnerability.

**Working Flow**
![flow-chart-4](flow-chart-4.png)

**Example**
```javascript
const div = document.querySelector("#output");

// SAFE — text only, no HTML parsing
div.textContent = userInput;  // even if userInput = "<script>alert(1)</script>"
// → displays literal text: <script>alert(1)</script>

// DANGEROUS — parses HTML, executes scripts
div.innerHTML = userInput;    // NEVER use with user data
// → could execute malicious scripts

// SAFE use of innerHTML — with trusted static content
div.innerHTML = `
  <h3>${sanitize(title)}</h3>
  <p>${sanitize(description)}</p>
`;

// Reading content
div.textContent  // → plain text, no tags
div.innerHTML    // → full HTML string including child elements
```

---

### 5. classList and Attributes

**Theory**: `classList` is the modern, safe way to manage CSS classes. It has four methods: `add`, `remove`, `toggle`, and `contains`. For HTML attributes (`href`, `src`, `data-*`), use `setAttribute`/`getAttribute`.

**Working Flow**
![flow-chart-5](flow-chart-5.png)

**Example**
```javascript
const btn = document.querySelector("#btn");
const menu = document.querySelector("#menu");

// Toggle menu visibility on button click
btn.addEventListener("click", () => {
  menu.classList.toggle("open");       // add/remove "open"
  btn.classList.toggle("active");      // highlight active button

  const isOpen = menu.classList.contains("open");
  btn.textContent = isOpen ? "Close" : "Open";
});

// Attributes
const link = document.querySelector("a");
link.setAttribute("href", "https://example.com");
link.setAttribute("target", "_blank");
link.setAttribute("rel", "noopener");

console.log(link.getAttribute("href")); // "https://example.com"

// data-* attributes via dataset
const card = document.querySelector(".card");
card.setAttribute("data-user-id", "42");
console.log(card.dataset.userId);       // "42"  (camelCase access)
```

**Output**
```
getAttribute("href")  → "https://example.com"
card.dataset.userId   → "42"
classList after toggle → "open" added or removed
```

---

### 6. Event Listeners — Responding to User Actions

**Theory**: `addEventListener` attaches a function to an element that runs when a specific event occurs. You can attach multiple listeners to the same element. Always use `addEventListener` over inline HTML event handlers (`onclick="..."`) for separation of concerns.

**Working Flow**
![flow-chart-6](flow-chart-6.png)

**Example**
```javascript
// Click event
const btn = document.querySelector("#submit-btn");
btn.addEventListener("click", (event) => {
  console.log("Clicked:", event.target.id);
  event.target.disabled = true; // prevent double-click
});

// Input event — real-time character counter
const textarea = document.querySelector("textarea");
const counter = document.querySelector("#counter");
textarea.addEventListener("input", (e) => {
  const length = e.target.value.length;
  counter.textContent = `${length}/200`;
  counter.classList.toggle("warning", length > 180);
});

// Keydown — submit on Enter
const searchInput = document.querySelector("#search");
searchInput.addEventListener("keydown", (e) => {
  if (e.key === "Enter") {
    performSearch(e.target.value);
  }
  if (e.key === "Escape") {
    e.target.value = "";
  }
});

// Form submit — prevent default page reload
const form = document.querySelector("form");
form.addEventListener("submit", async (e) => {
  e.preventDefault();  // stop page from reloading
  const formData = new FormData(e.target);
  const data = Object.fromEntries(formData);
  await submitToServer(data);
});
```

---

### Real-World Example — Dynamic List with Form Validation

```javascript
// HTML assumed:
// <input id="item-input" placeholder="Add item...">
// <button id="add-btn">Add</button>
// <ul id="item-list"></ul>

const input = document.querySelector("#item-input");
const addBtn = document.querySelector("#add-btn");
const list = document.querySelector("#item-list");

function addItem(text) {
  if (!text.trim()) {
    input.classList.add("error");
    return;
  }
  input.classList.remove("error");

  const li = document.createElement("li");
  li.innerHTML = `
    <span>${text}</span>
    <button class="delete-btn" data-text="${text}">Remove</button>
  `;

  // Delete this item when remove clicked
  li.querySelector(".delete-btn").addEventListener("click", () => {
    li.remove();
  });

  list.appendChild(li);
  input.value = "";
  input.focus();
}

addBtn.addEventListener("click", () => addItem(input.value));
input.addEventListener("keydown", (e) => {
  if (e.key === "Enter") addItem(input.value);
});
```

---

[View Interview Questions](./interview.md)
