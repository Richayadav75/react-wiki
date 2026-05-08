# Date Methods — Interview Questions

---

**1. Why are months zero-indexed in JavaScript's `Date` object?**

JavaScript inherited this from Java's `java.util.Date`. Months go from `0` (January) to `11` (December). This is one of the most common JavaScript bugs.

```javascript
const d = new Date("2024-03-15");
d.getMonth();     // → 2  (March — NOT 3!)
d.getMonth() + 1; // → 3  (correct for display)

// Creating a date — month is also 0-indexed
const jan = new Date(2024, 0, 15); // January 15
const dec = new Date(2024, 11, 25); // December 25

// Always show month + 1 to users
const display = `${d.getDate()}/${d.getMonth() + 1}/${d.getFullYear()}`;
// → "15/3/2024"
```

---

**2. What is the difference between `getDate()`, `getDay()`, and `getTime()`?**

| Method | Returns | Range/Example |
|--------|---------|---------------|
| `getDate()` | Day of the **month** | 1 to 31 |
| `getDay()` | Day of the **week** | 0 (Sun) to 6 (Sat) |
| `getTime()` | Milliseconds since Epoch | e.g. 1710494445000 |

```javascript
const d = new Date("2024-03-15"); // Friday, March 15

d.getDate();  // → 15     (15th of the month)
d.getDay();   // → 5      (Friday — 0=Sun,1=Mon,...,5=Fri)
d.getTime();  // → 1710460800000 (timestamp)

const days = ["Sun","Mon","Tue","Wed","Thu","Fri","Sat"];
days[d.getDay()]; // → "Fri"
```

---

**3. What is the Unix Epoch and why does JavaScript use it?**

The Unix Epoch is January 1, 1970 00:00:00 UTC. JavaScript (and most programming languages) stores dates as the number of milliseconds since this moment. This makes date arithmetic simple — dates are just numbers.

```javascript
new Date(0); // → Thu Jan 01 1970 00:00:00 GMT
new Date(1000); // → 1 second after epoch

// Because dates are numbers, math works directly:
const d1 = new Date("2024-01-01");
const d2 = new Date("2024-12-31");
const daysBetween = (d2 - d1) / (1000 * 60 * 60 * 24);
console.log(daysBetween); // → 365
```

---

**4. What is the difference between `Date.now()` and `new Date().getTime()`?**

Both return the current timestamp in milliseconds, but `Date.now()` is the preferred approach — it is faster (no object creation) and more readable.

```javascript
Date.now();            // → 1710500000000  (static method, no object)
new Date().getTime();  // → 1710500000000  (creates Date, then reads time)

// Use case: measuring performance
const start = Date.now();
// ... code ...
console.log(`Elapsed: ${Date.now() - start}ms`);
```

`Date.now()` is better for performance measurement and timestamp operations. Use `new Date()` when you need to display or manipulate date components.

---

**5. How do you calculate the number of days between two dates?**

Subtract the timestamps and convert from milliseconds to days:

```javascript
function daysBetween(date1, date2) {
  const msPerDay = 24 * 60 * 60 * 1000;
  const diff = Math.abs(new Date(date2) - new Date(date1));
  return Math.round(diff / msPerDay);
}

daysBetween("2024-01-01", "2024-12-31"); // → 365
daysBetween("2024-03-01", "2024-03-15"); // → 14

// Days until a future date
function daysUntil(targetDate) {
  return daysBetween(new Date(), targetDate);
}
daysUntil("2025-01-01"); // → varies based on today
```

---

**6. How do you format a date to a human-readable string?**

Use `toLocaleDateString()` with locale and options:

```javascript
const d = new Date("2024-03-15T10:30:00");

d.toLocaleDateString("en-IN");
// → "15/3/2024"

d.toLocaleDateString("en-US", {
  weekday: "long", year: "numeric",
  month: "long", day: "numeric"
});
// → "Friday, March 15, 2024"

d.toLocaleTimeString("en-US", {
  hour: "2-digit", minute: "2-digit"
});
// → "10:30 AM"

d.toISOString();
// → "2024-03-15T05:00:00.000Z"  (UTC — good for APIs)
```

---

**7. How do you add days, months, or years to a date?**

Use setter methods — JavaScript handles overflow automatically (e.g., adding 31 days to March 1 gives April 1):

```javascript
const d = new Date("2024-01-15");

// Add days
d.setDate(d.getDate() + 30);       // → Feb 14, 2024

// Add months
d.setMonth(d.getMonth() + 3);      // → May 14, 2024

// Add years
d.setFullYear(d.getFullYear() + 1); // → May 14, 2025

// Cleaner approach — don't mutate original
function addDays(date, days) {
  const result = new Date(date);
  result.setDate(result.getDate() + days);
  return result;
}
addDays(new Date("2024-01-31"), 1);
// → Feb 1, 2024  (overflow handled automatically)
```

---

**8. How do you build a countdown timer in JavaScript?**

Calculate remaining time and break it into days/hours/minutes/seconds:

```javascript
function getCountdown(targetDate) {
  const remaining = new Date(targetDate) - Date.now();
  if (remaining <= 0) return null;

  return {
    days:    Math.floor(remaining / (1000 * 60 * 60 * 24)),
    hours:   Math.floor((remaining % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60)),
    minutes: Math.floor((remaining % (1000 * 60 * 60)) / (1000 * 60)),
    seconds: Math.floor((remaining % (1000 * 60)) / 1000)
  };
}

// Update every second
setInterval(() => {
  const t = getCountdown("2025-01-01");
  if (!t) { console.log("Expired!"); return; }
  console.log(`${t.days}d ${t.hours}h ${t.minutes}m ${t.seconds}s`);
}, 1000);
```

---

**9. What is the "month trap" and how do you avoid it?**

The month trap is forgetting that `getMonth()` returns 0-11 instead of 1-12. It causes off-by-one bugs in date display and creation.

```javascript
// WRONG — month trap in creation
const badDate = new Date(2024, 3, 15); // You think: April 15
// Actually: April 15 ✓ in this case — but...
const confusion = new Date(2024, 12, 1); // Month 12 → January 2025!

// WRONG — month trap in display
const d = new Date("2024-03-15");
console.log(`${d.getMonth()}/15/2024`); // → "2/15/2024" (wrong! shows 2 not 3)

// CORRECT
console.log(`${d.getMonth() + 1}/15/2024`); // → "3/15/2024"

// Safe display helper
function formatDate(date) {
  const d = new Date(date);
  return `${d.getDate().toString().padStart(2,'0')}/${
    (d.getMonth() + 1).toString().padStart(2,'0')}/${d.getFullYear()}`;
}
formatDate("2024-03-05"); // → "05/03/2024"
```

---

**10. How do you check if a given year is a leap year?**

```javascript
function isLeapYear(year) {
  return (year % 4 === 0 && year % 100 !== 0) || year % 400 === 0;
}

isLeapYear(2024); // → true   (divisible by 4)
isLeapYear(1900); // → false  (divisible by 100 but not 400)
isLeapYear(2000); // → true   (divisible by 400)

// Using Date trick — Feb 29 exists only in leap years
function isLeapYearDate(year) {
  return new Date(year, 1, 29).getDate() === 29;
}

// Days in February
function daysInFeb(year) {
  return isLeapYear(year) ? 29 : 28;
}
daysInFeb(2024); // → 29
daysInFeb(2023); // → 28
```
