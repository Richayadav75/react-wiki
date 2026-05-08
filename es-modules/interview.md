# ES Modules Interview Questions

---

**1. What is the difference between named exports and default exports?**

- **Named exports** — a file can have many. Must be imported using the exact name in `{}`. Good for utility files that expose multiple functions.
- **Default export** — only one per file. Imported without `{}` and can be given any name. Good for components, classes, or the single main thing a file provides.

```javascript
// Named exports
export const add = (a, b) => a + b;
export const PI = 3.14;

// Default export
export default function Button() { return "<button>"; }

// Importing both
import Button, { add, PI } from './file.js';
import MyButton from './file.js'; // default can use any name
```

---

**2. What is tree shaking and why does it only work with ESM?**

Tree shaking is a bundler optimization (Webpack, Vite, Rollup) that removes exports that are never imported anywhere — dead code that never executes. It works with ESM because:
- ESM `import`/`export` statements are static — evaluated at parse time before code runs.
- The bundler can statically trace which exports are actually used.
- CommonJS `require()` is dynamic — evaluated at runtime — so the bundler can't know in advance what's used.

```javascript
// math.js
export const add = ...;      // used
export const multiply = ...; // NOT imported anywhere → removed from bundle

// app.js
import { add } from './math.js';  // multiply tree-shaken away
```

---

**3. Can you use import inside an if block?**

Static `import` statements must appear at the top level of a module — never inside functions, conditionals, or loops. This is intentional: static imports allow the JavaScript engine to analyze dependencies before executing code.

For conditional loading, use **dynamic `import()`**, which is a function that returns a Promise:

```javascript
// INVALID — SyntaxError
if (isDev) {
  import logger from './logger.js'; // error!
}

// VALID — dynamic import
if (isDev) {
  const { default: logger } = await import('./logger.js');
  logger.enable();
}
```

---

**4. What does it mean that modules execute only once (singleton)?**

No matter how many files import the same module, the module body runs exactly once. All importers share the same exported bindings. This is the singleton pattern by default:

```javascript
// store.js
let state = { count: 0 };
export function increment() { state.count++; }
export function getCount() { return state.count; }
console.log("store.js initialized"); // runs once

// a.js imports store.js — body runs, state.count = 0
// b.js imports store.js — NO re-run, shares the same state
// if a.js calls increment(), b.js sees count = 1
```

---

**5. What is a barrel file (index.js) and what problem does it solve?**

A barrel file re-exports from multiple modules, providing a single entry point for a directory. It simplifies imports in consuming code:

```javascript
// Without barrel — messy imports
import { fetchUser } from '../api/users.js';
import { fetchPosts } from '../api/posts.js';
import { formatDate } from '../utils/format.js';

// With barrel (api/index.js re-exports everything)
import { fetchUser, fetchPosts } from '../api';
import { formatDate } from '../utils';
```

One downside: if the barrel imports everything, tree shaking may be less effective. Export only what's genuinely public.

---

**6. What is the difference between ESM and CommonJS (require)?**

| Feature | ESM | CommonJS |
|---|---|---|
| Syntax | `import` / `export` | `require()` / `module.exports` |
| Loading | Static (parsed before run) | Dynamic (evaluated at runtime) |
| Browser support | Native (no bundler needed) | Requires bundler |
| Top-level await | Supported | Not supported |
| Tree shaking | Yes — static analysis | No — dynamic |
| `this` at top level | `undefined` | `module.exports` object |

Use ESM for all new browser and Node.js projects. CommonJS is still common in older npm packages.

---

**7. What is import.meta and what is it used for?**

`import.meta` is an object available inside ES Modules containing module-specific metadata. The most useful property is `import.meta.url` — the absolute URL of the current module file.

```javascript
// In browser
console.log(import.meta.url); // "http://localhost:3000/src/app.js"

// In Node.js
console.log(import.meta.url); // "file:///Users/me/project/src/app.js"

// Resolve paths relative to current module (Node.js)
import { fileURLToPath } from 'url';
import { dirname, join } from 'path';
const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);
const configPath = join(__dirname, 'config.json');
```

---

**8. How do you handle circular imports in ESM?**

Circular imports (A imports B, B imports A) are technically allowed in ESM but can lead to `undefined` values if a binding is accessed before it's initialized. The engine resolves the graph in phases — bindings exist but may not yet have their values.

Best practice: avoid circular imports by extracting shared code into a third module that both A and B import.

```javascript
// shared.js
export const BASE_URL = "https://api.example.com";

// users.js imports shared.js  ← no cycle
// posts.js imports shared.js  ← no cycle
// users.js and posts.js don't import each other
```

---

**9. What is the difference between dynamic import() and a regular import statement?**

| | Static import | Dynamic import() |
|---|---|---|
| When evaluated | Parse time (before code runs) | Runtime (when line executes) |
| Location | Top level only | Anywhere (if, function, click handler) |
| Return value | Bound module namespace | Promise resolving to module namespace |
| Use case | Primary dependencies | Lazy loading, code splitting, conditionals |

```javascript
// static — always loaded upfront
import { render } from './renderer.js';

// dynamic — loaded only when function is called
async function openModal() {
  const { Modal } = await import('./Modal.js');
  new Modal().open();
}
```

---

**10. Are ES Modules in strict mode by default?**

Yes. All ES Modules run in strict mode automatically — you don't need `"use strict"` at the top. This means:
- No implicit global variables (`x = 5` without `let/const/var` throws ReferenceError)
- `this` at the top level is `undefined` (not `window`)
- `with` statement is forbidden
- Duplicate parameter names are not allowed
- Octal literals (`0777`) are not allowed

This is one reason ESM encourages cleaner code habits compared to sloppy-mode scripts.
