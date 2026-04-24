# Closures Interview Questions

1. **What is a closure?**
   - A function that "remembers" its lexical scope even when the function is executed outside that scope.

2. **Give a practical use case for closures.**
   - Data privacy (private variables), function factories, and maintaining state in asynchronous callbacks.

3. **Can closures cause memory leaks?**
   - Yes, if a closure retains a large object that is no longer needed, it can prevent that object from being garbage collected.
