- Category: DOM Manipulation
- Difficulty: Intermediate
- Related: events, dom, bom

### Event Delegation — One Listener to Rule Them All
Event delegation is a pattern where you attach a **single event listener to a parent element** instead of attaching individual listeners to each child. When a child fires an event, it bubbles up to the parent, which then checks what was clicked and handles it.

**Analogy**
A post office counter. Instead of hiring one staff member for every possible sender who might walk in, one receptionist (parent listener) handles all arrivals. When anyone walks in (event bubbles up), the receptionist checks their ID (event.target) and routes the package (handles the action) to the right department.

---

### 1. The Problem — Listeners on Every Child

**Theory**
When you have a list of items and attach `addEventListener` to each one, you create N listeners — one per item. This wastes memory, and more critically, any items added to the DOM dynamically after the listeners are attached won't have listeners at all.

**Working Flow**

![flow-chart](flow-chart.png)

**Example**
```javascript
// ❌ Without delegation — listener on every button
const buttons = document.querySelectorAll('.delete-btn');

buttons.forEach(btn => {
  btn.addEventListener('click', function() {
    console.log('Delete:', this.dataset.id);
  });
});

// Problems:
// 1. N listeners created (memory)
// 2. Dynamically added buttons have NO listener
const newBtn = document.createElement('button');
newBtn.className = 'delete-btn';
newBtn.dataset.id = '999';
document.querySelector('.list').appendChild(newBtn);
// ^ this button has NO click listener!
```

**Output**
```
// Click on original button:
Delete: 1   ← works

// Click on dynamically added button:
(silence)   ← no listener was attached to it
```

---

### 2. With Event Delegation — One Parent Listener

**Theory**
Place one listener on the parent container. Use `event.target` to identify which child was clicked. This works for existing AND future dynamically added children automatically.

**Working Flow**

![flow-chart-2](flow-chart-2.png)

**Example**
```javascript
const list = document.querySelector('.list');

// ✅ One listener on the parent
list.addEventListener('click', function(event) {
  const btn = event.target.closest('.delete-btn');
  if (!btn) return;   // click was somewhere else in the list

  const id = btn.dataset.id;
  console.log('Delete:', id);

  // Remove the parent list item
  btn.closest('li').remove();
});

// Dynamically add a new item — listener already covers it!
function addItem(id, text) {
  const li  = document.createElement('li');
  li.innerHTML = `
    <span>${text}</span>
    <button class="delete-btn" data-id="${id}">Delete</button>
  `;
  list.appendChild(li);
}

addItem(1, 'Buy groceries');
addItem(2, 'Write code');
addItem(3, 'Go for a run');
```

**Output**
```
// Click Delete on "Buy groceries":
Delete: 1   ← works

// Click Delete on dynamically added item:
Delete: 3   ← works too! Delegation handles future items
```

---

### 3. event.target vs event.currentTarget

**Theory**
Understanding the difference is critical for delegation:
- `event.target` — the element that was **actually clicked** (the source)
- `event.currentTarget` — the element that has the **listener attached** (always the parent in delegation)

**Working Flow**

![flow-chart-3](flow-chart-3.png)

**Example**
```javascript
const nav = document.querySelector('nav');

nav.addEventListener('click', function(event) {
  console.log('target:',        event.target.tagName);
  console.log('currentTarget:', event.currentTarget.tagName);
});
```

**Output**
```
// User clicks a <button> inside <nav>:
target:         BUTTON   ← what was clicked
currentTarget:  NAV      ← where the listener lives

// User clicks a <span> inside a <button> inside <nav>:
target:         SPAN     ← innermost element clicked
currentTarget:  NAV

// This is why we use .closest() — target might be a child of the real button
```

**Why `.closest()` matters:**
```javascript
// ❌ Might fail — user could click the icon inside the button
if (event.target.classList.contains('delete-btn')) { ... }

// ✅ Works regardless of which child inside the button was clicked
const btn = event.target.closest('.delete-btn');
if (btn) { ... }
```

---

### 4. data-* Attributes — Identifying the Right Child

**Theory**
Use HTML `data-*` attributes to store metadata on child elements. The parent listener reads these attributes from `event.target` to know which item to act on — without needing separate closures per item.

**Working Flow**

![flow-chart-4](flow-chart-4.png)

**Example**
```javascript
// HTML structure
/*
  <ul id="product-list">
    <li data-id="1" data-price="800">
      <span>Phone</span>
      <button class="btn-add">Add to Cart</button>
      <button class="btn-view">View Details</button>
    </li>
    <li data-id="2" data-price="200">
      <span>Watch</span>
      <button class="btn-add">Add to Cart</button>
      <button class="btn-view">View Details</button>
    </li>
  </ul>
*/

document.querySelector('#product-list').addEventListener('click', function(e) {
  // Find the clicked button
  const btn  = e.target.closest('button');
  if (!btn) return;

  // Find the parent <li> for metadata
  const item = btn.closest('li');
  const id    = item.dataset.id;
  const price = item.dataset.price;

  if (btn.classList.contains('btn-add')) {
    console.log(`Added to cart: id=${id}, price=₹${price}`);
  }

  if (btn.classList.contains('btn-view')) {
    console.log(`Viewing details for: id=${id}`);
  }
});
```

**Output**
```
// Click "Add to Cart" on Phone:
Added to cart: id=1, price=₹800

// Click "View Details" on Watch:
Viewing details for: id=2
```

---

### 5. Tab Navigation — Real-World Pattern

**Theory**
Delegating tab clicks to a container is a clean, common pattern. One listener handles all tab buttons, finds the active one, and swaps content — even if tabs are added later.

**Working Flow**

![flow-chart-5](flow-chart-5.png)

**Example**
```javascript
const tabBar     = document.querySelector('.tab-bar');
const tabContent = document.querySelector('.tab-content');

tabBar.addEventListener('click', function(event) {
  const tab = event.target.closest('[data-tab]');
  if (!tab) return;

  // Remove active from all tabs
  tabBar.querySelectorAll('[data-tab]').forEach(t => {
    t.classList.remove('active');
  });

  // Mark clicked tab as active
  tab.classList.add('active');

  // Show the corresponding panel
  const panelId = tab.dataset.tab;
  tabContent.querySelectorAll('.panel').forEach(panel => {
    panel.hidden = panel.id !== panelId;
  });

  console.log('Switched to:', panelId);
});
```

**Output**
```
// Click "Profile" tab:
Switched to: profile
← profile panel visible, others hidden

// Click "Settings" tab:
Switched to: settings
← settings panel visible, others hidden
```

---

### 6. When NOT to Use Delegation

**Theory**
Delegation is powerful but not always the right tool. Avoid it when:
- A child calls `stopPropagation()` — the event never reaches the parent listener
- You need `mouseenter`/`mouseleave` — these don't bubble (use `mouseover`/`mouseout` instead)
- The parent is very high in the DOM tree and the target check is complex

**Working Flow**

![flow-chart-6](flow-chart-6.png)

**Example**
```javascript
// ❌ stopPropagation breaks delegation
document.querySelector('.child').addEventListener('click', e => {
  e.stopPropagation(); // event never reaches parent → delegation fails
});

// ❌ mouseenter does NOT bubble
list.addEventListener('mouseenter', e => {
  // Only fires when entering .list itself — not child <li> elements
});

// ✅ Use mouseover instead — it bubbles
list.addEventListener('mouseover', e => {
  const li = e.target.closest('li');
  if (li) li.classList.add('hovered');
});
list.addEventListener('mouseout', e => {
  const li = e.target.closest('li');
  if (li) li.classList.remove('hovered');
});
```

| Use delegation | Use direct listeners |
| :--- | :--- |
| Large lists of similar items | Complex unique per-element behaviour |
| Dynamically added elements | Events that don't bubble (mouseenter) |
| Memory efficiency needed | Child stops propagation |

---

[View Interview Questions](./interview.md)
