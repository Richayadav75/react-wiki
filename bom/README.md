- Category: Core Concepts
- Difficulty: Intermediate
- Related: dom

### What is the BOM?
The BOM (Browser Object Model) allows JavaScript to "talk to" the browser. It is centered around the `window` object.

---

### 1. The Window Object
**Theory**: The `window` object represents the browser's window. All global JavaScript objects, functions, and variables automatically become members of the window object.

**Working Flow**
```text
[ Browser Window ]
   ├── document (DOM)
   ├── location (URL info)
   └── navigator (Browser info)
```

**Key Features**:
- **window.innerHeight / innerWidth**: Gets the viewport size.
- **window.location**: Gets the current URL and allows you to redirect the user.
- **window.navigator**: Information about the user's browser/OS.

**Step-by-Step Example**:
```javascript
// Step 1: Get the height of the browser window
const height = window.innerHeight;
console.log(`Window height is: ${height}px`);

// Step 2: Get the current URL
const currentUrl = window.location.href;
console.log(currentUrl);

// Step 3: Redirect the user to a new page
// window.location.assign("https://geeksforgeeks.org");
```
