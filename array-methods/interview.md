# Array Methods Interview Questions

1. **Difference between map() and forEach()?**
   - `map()` returns a new array with transformed elements. `forEach()` iterates over the array but returns `undefined` (used for side effects).

2. **How does reduce() work?**
   - It reduces an array to a single value by executing a reducer function on each element, passing the accumulated result from the previous iteration.

3. **Which methods mutate the original array?**
   - `push`, `pop`, `shift`, `unshift`, `splice`, `sort`, `reverse`. Methods like `map`, `filter`, and `slice` return new arrays.
