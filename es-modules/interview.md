# ES Modules Interview Questions

1. **What is the difference between Named and Default exports?**
   - A module can have multiple named exports but only one default export. Named exports must be imported with their exact names in curly braces.

2. **Can you import ES modules in a Node.js environment?**
   - Yes, by using the `.mjs` extension or setting `"type": "module"` in `package.json`.

3. **What are the advantages of static imports?**
   - Enables tree-shaking (removing unused code) and static analysis by tools/compilers.
