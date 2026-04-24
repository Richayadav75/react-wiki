- Category: JavaScript
- Difficulty: Beginner
- Related: ES6 Features

# ES Modules (ESM)

ES Modules are the official standard format to package JavaScript code for reuse. They use `import` and `export` statements to share functionality between files.

## Exporting

### Named Exports
You can have multiple named exports per file.
```javascript
// math.js
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;
```

### Default Export
Each file can have only one default export.
```javascript
// logger.js
export default function log(message) {
  console.log(message);
}
```

## Importing

```javascript
import log from './logger.js'; // Default import
import { add, subtract } from './math.js'; // Named imports

log(add(5, 3)); // 8
```

## Practice: Dynamic Imports
Dynamic imports allow you to load modules on demand using `import()`, which returns a promise.

```javascript
async function loadMath() {
  const math = await import('./math.js');
  console.log(math.add(10, 5));
}

loadMath();
```

## Interview Questions
1. **What is the difference between Named and Default exports?**
   Named exports require braces and must match the name exactly. Default exports can be imported with any name and don't use braces.
2. **What are the benefits of ES Modules over CommonJS?**
   ESM supports static analysis, which enables "tree-shaking" (removing unused code). It is also the browser-native standard.

---

[View Interview Questions](./interview.md)
