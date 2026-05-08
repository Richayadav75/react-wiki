# ES Modules (ESM) Interview Questions

1. **What is the difference between Named and Default exports?**
   - **Named Exports**: You can have multiple per file. They must be imported with their exact name inside curly braces `{}`.
   - **Default Export**: Only one per file. It can be imported with any name and does not use curly braces.

2. **What are the benefits of ES Modules over CommonJS?**
   - ESM supports **static analysis**, which allows tools to perform **tree-shaking** (removing unused code).
   - ESM is the official browser standard, while CommonJS was designed for Node.js servers.

3. **How do you rename an import?**
   - Using the `as` keyword: `import { originalName as newName } from './file.js';`.

4. **What is a "Namespace Import"?**
   - It is when you import all exports from a module into a single object: `import * as Utils from './utils.js';`.

5. **What are Dynamic Imports?**
   - They allow you to load modules asynchronously on demand (lazy loading) using the `import()` function, which returns a promise. This is crucial for optimizing large applications.

6. **Can you use `import` inside a conditional (like an `if` block)?**
   - Regular `import` statements must be at the top level. To import inside a block or function, you must use **Dynamic Imports** (`import()`).

7. **What is "Tree Shaking"?**
   - It is a build-step optimization that removes unused code from your final bundle. It only works with ES Modules because their static structure allows the bundler to know exactly what is being used at compile-time.
