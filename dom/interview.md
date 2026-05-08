# DOM Interview Questions

---

**1. What is the difference between innerHTML and textContent?**

- `textContent` sets or gets plain text. All HTML tags are treated as literal characters — safe against XSS.
- `innerHTML` parses the string as HTML and renders it. Never use with untrusted user input — a malicious string like `<img src=x onerror="alert(1)">` executes JavaScript.

```javascript
div.textContent = "<b>bold</b>";
// Renders: <b>bold</b>  (literal text, shows the tags)

div.innerHTML = "<b>bold</b>";
// Renders: bold  (HTML parsed, text is bold)

// DANGER — user input
div.innerHTML = userInput;  // XSS risk
div.textContent = userInput; // safe — always use for user data
```

---

**2. What is the difference between querySelector and getElementById?**

- `getElementById("id")` — selects by exact ID string, fastest, returns `null` if not found. Only works for IDs.
- `querySelector("#id")` — accepts any CSS selector (class, tag, attribute, pseudo-class). Returns the first match or `null`. More flexible but slightly slower.
- `querySelectorAll` — returns all matches as a static NodeList.

Use `getElementById` when you only need an ID lookup. Use `querySelector` for complex selections.

---

**3. What is the difference between a NodeList and an HTMLCollection?**

| | NodeList | HTMLCollection |
|---|---|---|
| Returned by | `querySelectorAll`, `childNodes` | `getElementsByTagName`, `getElementsByClassName`, `children` |
| Live? | Static (snapshot when called) | Live (updates as DOM changes) |
| `forEach`? | Yes | No |
| `map/filter`? | No — convert with `Array.from()` | No — convert with `Array.from()` |

```javascript
const nodes = document.querySelectorAll("li"); // static NodeList
nodes.forEach(n => console.log(n.textContent)); // works

const collection = document.getElementsByTagName("li"); // live HTMLCollection
Array.from(collection).forEach(n => console.log(n.textContent));
```

---

**4. What does event.preventDefault() do?**

It cancels the browser's default action for that event — without stopping the event from propagating. Common uses:
- Form submit: prevents page reload
- Link click: prevents navigation
- Keydown on Enter in textarea: prevents line break

```javascript
form.addEventListener("submit", (e) => {
  e.preventDefault(); // no page reload
  const data = new FormData(e.target);
  submitAjax(data);
});

link.addEventListener("click", (e) => {
  e.preventDefault(); // no navigation
  showModal();
});
```

---

**5. What is the difference between event.target and event.currentTarget?**

- `event.target` — the actual element that was clicked/triggered (could be a child).
- `event.currentTarget` — the element the listener is attached to (always consistent).

```javascript
// HTML: <ul id="list"><li>Item 1</li></ul>
document.querySelector("#list").addEventListener("click", (e) => {
  console.log(e.target);        // <li>Item 1</li>  (what was clicked)
  console.log(e.currentTarget); // <ul id="list">   (where listener lives)
});
```

This difference is the basis of **event delegation** — attach one listener to a parent to handle clicks on many children.

---

**6. Why is event delegation better than attaching listeners to every element?**

Attaching a listener to each of 100 list items creates 100 listeners — memory expensive and requires re-attaching when items are dynamically added. Event delegation attaches one listener to the parent — child events bubble up to it.

```javascript
// Bad — 100 listeners, breaks for dynamically added items
document.querySelectorAll("li").forEach(li => {
  li.addEventListener("click", handleClick);
});

// Good — one listener, works for current and future items
document.querySelector("ul").addEventListener("click", (e) => {
  if (e.target.tagName === "LI") {
    handleClick(e.target);
  }
});
```

---

**7. What is a DocumentFragment and why is it useful?**

A `DocumentFragment` is an invisible container — a lightweight DOM node that lives in memory, not in the page. You can build a large structure inside it, then append it to the DOM in one operation. This causes only one reflow/repaint instead of one per element.

```javascript
const fragment = document.createDocumentFragment();

["Apple", "Banana", "Mango"].forEach(name => {
  const li = document.createElement("li");
  li.textContent = name;
  fragment.appendChild(li); // no DOM reflow yet
});

document.querySelector("ul").appendChild(fragment); // one reflow
```

---

**8. What is the difference between appendChild and append?**

- `appendChild(node)` — older API, accepts only a single Node, returns the appended node.
- `append(...items)` — modern API, accepts multiple nodes AND strings (auto-converts strings to text nodes), returns `undefined`.

```javascript
const ul = document.querySelector("ul");

// appendChild — single node only
ul.appendChild(document.createElement("li"));

// append — multiple, mixed types
ul.append(
  document.createElement("li"),
  document.createElement("li"),
  "direct text" // becomes text node
);
```

---

**9. How do you efficiently update a list of DOM elements?**

Three strategies:
1. **DocumentFragment** — batch-build then single insert (best for large lists).
2. **innerHTML replacement** — replace entire inner HTML (fast, but loses event listeners on children).
3. **Targeted updates** — only update changed elements (best for small, frequent updates).

For very large lists (thousands of items), use **virtual scrolling** — only render visible items.

---

**10. What is the difference between setAttribute and direct property assignment?**

- `setAttribute("class", "active")` — sets the HTML attribute (string-based).
- `element.className = "active"` — sets the DOM property (may differ from attribute).
- For most standard attributes they sync, but for `checked`, `value`, and custom `data-*`, behavior can differ.

Prefer `.classList` for classes, `.value` for inputs, and `setAttribute` for custom/non-standard attributes:

```javascript
input.setAttribute("disabled", "");  // sets attribute (HTML)
input.disabled = true;               // sets property (JS) — preferred for boolean attributes

el.setAttribute("data-id", "42");   // custom data attribute
el.dataset.id;                       // read via dataset → "42"
```
