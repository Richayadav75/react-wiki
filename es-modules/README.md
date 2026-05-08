- Category: JavaScript
- Difficulty: Intermediate
- Related: es6-features, async-await

### ES Modules — Organizing JavaScript into Files

Before modules, all JavaScript shared a single global scope — every file's variables could collide with every other file's. **ES Modules (ESM)** solve this by giving each file its own scope. You explicitly choose what to share (`export`) and what to use from other files (`import`). The result is maintainable, tree-shakable, reusable code.

**Analogy**
Think of each module as a specialist store on a shopping street. The `math.js` store sells mathematical tools. You walk in and pick exactly what you need (`import { add } from './math.js'`). The store doesn't dump everything on the street — it only shows its shopfront (`export`). Other stores can't touch what's in the back room (private variables).

---

### 1. Named Exports — Multiple Exports per File

**Theory**: Named exports let a file share multiple values. They must be imported using their exact name inside curly braces `{}`. Best used when a module provides a collection of related utilities.

**Working Flow**
![flow-chart](flow-chart.png)

**Example**
```javascript
// math.js
export const PI = 3.14159;

export function add(a, b) {
  return a + b;
}

export function multiply(a, b) {
  return a * b;
}

// Private — not exported, not accessible from outside
function internalHelper() {
  return "only used inside math.js";
}
```

```javascript
// app.js
import { PI, add, multiply } from './math.js';

console.log(PI);           // 3.14159
console.log(add(2, 3));    // 5
console.log(multiply(4, 5)); // 20

// Rename on import with 'as'
import { multiply as times } from './math.js';
console.log(times(3, 4)); // 12
```

**Output**
```
PI           → 3.14159
add(2, 3)    → 5
multiply(4,5)→ 20
times(3, 4)  → 12
```

---

### 2. Default Export — One Primary Export per File

**Theory**: Each file can have at most one default export — the "main thing" the file provides. It is imported without curly braces and can be given any name by the importer. Best for components, classes, or the single primary export of a file.

**Working Flow**
![flow-chart-2](flow-chart-2.png)

**Example**
```javascript
// logger.js — default export
export default function log(message, level = "info") {
  const timestamp = new Date().toISOString();
  console.log(`[${level.toUpperCase()}] ${timestamp}: ${message}`);
}

// Also export a named constant alongside the default
export const LOG_LEVELS = ["info", "warn", "error"];
```

```javascript
// app.js
import log from './logger.js';                       // default — any name
import myLog from './logger.js';                     // also valid — same export
import log, { LOG_LEVELS } from './logger.js';       // default + named

log("App started");                    // [INFO] 2026-05-08T...: App started
log("Disk full", "warn");              // [WARN] 2026-05-08T...: Disk full
console.log(LOG_LEVELS);              // ["info", "warn", "error"]
```

**Output**
```
[INFO] 2026-05-08T10:00:00.000Z: App started
[WARN] 2026-05-08T10:00:00.001Z: Disk full
LOG_LEVELS → ["info", "warn", "error"]
```

---

### 3. Import All as Namespace

**Theory**: Use `import * as Name` to import all named exports of a module under a single namespace object. Useful when you want to use many things from a module but don't want a long import list.

**Working Flow**
![flow-chart-3](flow-chart-3.png)

**Example**
```javascript
// utils/string.js
export function capitalize(str) {
  return str.charAt(0).toUpperCase() + str.slice(1);
}
export function truncate(str, maxLen) {
  return str.length > maxLen ? str.slice(0, maxLen) + "..." : str;
}
export function slugify(str) {
  return str.toLowerCase().replace(/\s+/g, "-");
}
```

```javascript
// app.js
import * as StringUtils from './utils/string.js';

console.log(StringUtils.capitalize("hello world")); // "Hello world"
console.log(StringUtils.truncate("A long sentence here", 10)); // "A long sen..."
console.log(StringUtils.slugify("Hello World"));   // "hello-world"
```

**Output**
```
capitalize("hello world") → "Hello world"
truncate("A long sentence here", 10) → "A long sen..."
slugify("Hello World")    → "hello-world"
```

---

### 4. Re-exporting — Barrel Files

**Theory**: A barrel file (`index.js`) re-exports from multiple modules, giving consumers a single clean import point. Instead of importing from 10 different paths, they import from one directory.

**Working Flow**
![flow-chart-4](flow-chart-4.png)

**Example**
```javascript
// utils/index.js — barrel file
export { capitalize, truncate, slugify } from './string.js';
export { add, multiply, PI } from './math.js';
export { formatDate, isWeekend } from './date.js';
export { default as Logger } from './logger.js'; // re-export default as named
```

```javascript
// app.js — clean single import
import { add, capitalize, formatDate, Logger } from './utils';

add(1, 2);              // 3
capitalize("hello");    // "Hello"
formatDate(new Date()); // "2026-05-08"
```

---

### 5. Dynamic Import — Code Splitting and Lazy Loading

**Theory**: Static `import` statements are evaluated at load time — all modules load upfront. Dynamic `import()` is a function that loads a module on demand, returning a Promise. This enables code splitting: only load code when it's actually needed, reducing initial bundle size.

**Working Flow**
![flow-chart-5](flow-chart-5.png)

**Example**
```javascript
// Load heavy library only when user opens the chart tab
const chartBtn = document.querySelector("#show-chart");
chartBtn.addEventListener("click", async () => {
  // Module loads only when button is clicked
  const { renderChart } = await import('./chart.js');
  renderChart(salesData);
});

// Route-based code splitting in a SPA
async function navigateTo(route) {
  let pageModule;

  switch (route) {
    case "/":
      pageModule = await import('./pages/Home.js');
      break;
    case "/settings":
      pageModule = await import('./pages/Settings.js'); // loaded lazily
      break;
    case "/admin":
      pageModule = await import('./pages/Admin.js');   // loaded lazily
      break;
  }

  pageModule.default.render(document.querySelector("#app"));
}

// Feature detection — load polyfill only if needed
if (!window.IntersectionObserver) {
  await import('./polyfills/intersection-observer.js');
}
```

---

### 6. Module Behavior — Singleton, Strict Mode, import.meta

**Theory**: Modules have three important automatic behaviors:
1. **Strict mode is always on** — no implicit globals, `this` is `undefined` at top level.
2. **Executed once (singleton)** — even if imported by 10 files, the module body runs once. The same exported values are shared by all importers.
3. **import.meta** — an object containing module-specific metadata, most importantly `import.meta.url` (the current module's URL).

**Working Flow**
![flow-chart-6](flow-chart-6.png)

**Example**
```javascript
// counter.js — singleton behavior
let count = 0;

export function increment() { count++; }
export function getCount() { return count; }

console.log("counter.js initialized"); // prints only ONCE
```

```javascript
// a.js
import { increment, getCount } from './counter.js';
increment();
console.log("A sees:", getCount()); // A sees: 1

// b.js
import { increment, getCount } from './counter.js';
increment();
console.log("B sees:", getCount()); // B sees: 2 (shared state!)
```

```javascript
// import.meta
console.log(import.meta.url);
// → "file:///Users/me/project/app.js" (in Node)
// → "http://localhost:3000/app.js"    (in browser)

// Dynamic import relative to current module
const data = await import(import.meta.resolve('./data.json'));
```

**Output**
```
counter.js initialized   ← only printed once (module runs once)
A sees: 1
B sees: 2               ← shared state proves singleton behavior
```

---

### 7. ESM vs CommonJS — Key Differences

**Theory**: Node.js historically used CommonJS (`require`/`module.exports`). Browsers and modern Node.js use ES Modules. Understanding both is essential for working with npm packages and Node.js environments.

**Working Flow**
![flow-chart-7](flow-chart-7.png)

**Example**
```javascript
// CommonJS (Node.js — .cjs or .js without "type":"module")
const { add, PI } = require('./math');
const express = require('express');
module.exports = { greet: () => "hello" };

// ES Modules (browser or Node.js with "type":"module" in package.json)
import { add, PI } from './math.js';
import express from 'express';
export function greet() { return "hello"; }

// Dynamic require in CJS (works)
const lib = require(condition ? './a' : './b');

// Dynamic import in ESM (works)
const lib = await import(condition ? './a.js' : './b.js');

// Cannot use require in ESM — use createRequire if needed
import { createRequire } from 'module';
const require = createRequire(import.meta.url);
const pkg = require('./package.json');
```

---

### Real-World Example — Feature Module Organization

```
src/
  api/
    index.js       ← barrel: exports fetchUser, fetchPosts, fetchComments
    users.js       ← named exports: fetchUser, createUser, updateUser
    posts.js       ← named exports: fetchPosts, createPost
  utils/
    index.js       ← barrel
    format.js      ← formatDate, formatCurrency, formatNumber
    validate.js    ← validateEmail, validatePhone
  components/
    Button.js      ← default export: Button component
    Modal.js       ← default export: Modal component
```

```javascript
// Clean imports in any file
import { fetchUser, createPost } from '../api';
import { formatDate, validateEmail } from '../utils';
import Button from '../components/Button';

async function createUserPost(userId, postData) {
  const user = await fetchUser(userId);
  const post = await createPost({ ...postData, author: user.name });
  return post;
}
```

---

[View Interview Questions](./interview.md)
