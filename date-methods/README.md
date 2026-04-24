- Category: JavaScript
- Difficulty: Beginner
- Related: Fundamentals

### What are Date Methods?
The `Date` object is used to work with dates and times. You create a new date instance and then use methods to get or set specific parts of the time.

---

### 1. Creating and Getting Dates
**Theory**: A Date object captures a specific moment in time. You can extract the year, month, day, or time from it.

**Working Flow**
```text
[ new Date() ] ---> ( Extract ) ---> [ Year: 2024, Month: 0 (Jan), Day: 15 ]
```

**Key Methods**:
- **getFullYear()**: Gets the 4-digit year.
- **getMonth()**: Gets the month (0-11, where 0 is January!).
- **getDate()**: Gets the day of the month (1-31).
- **getDay()**: Gets the day of the week (0-6, where 0 is Sunday).

**Step-by-Step Example**:
```javascript
// Step 1: Create a Date object for right now
const now = new Date();

// Step 2: Extract specific parts
const year = now.getFullYear();
const month = now.getMonth(); // Note: January is 0!
const date = now.getDate();

console.log(`Today is ${year}-${month + 1}-${date}`);
```

---

[View Interview Questions](./interview.md)
