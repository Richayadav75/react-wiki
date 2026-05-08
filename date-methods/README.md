- Category: JavaScript
- Difficulty: Beginner
- Related: variables, functions, math-methods

### JavaScript Date Methods — Working with Time
JavaScript's `Date` object represents a single moment in time. Internally it stores everything as a **timestamp** — the number of milliseconds since January 1, 1970 00:00:00 UTC (the Unix Epoch). All date operations are math on this single number.

**Analogy**
Think of the Unix Epoch as the odometer reset — the car started at 0 km on Jan 1, 1970. Every tick of the clock adds more milliseconds. `Date.now()` reads the current odometer value. `new Date()` is a dashboard that converts that number into human-readable year/month/day/time.

---

### 1. Creating Date Objects
**Theory**: There are four ways to create a `Date`. Understanding each helps you convert between timestamps, ISO strings, and human-readable components.

**Working Flow**
![flow-chart](flow-chart.png)

**Example**
```javascript
// Current date/time
const now = new Date();
console.log(now); // → Fri Mar 15 2024 10:30:00 GMT+0530 (IST)

// From timestamp (milliseconds since epoch)
const fromMs = new Date(0);
console.log(fromMs); // → Thu Jan 01 1970 00:00:00 GMT+0000

const fromTs = new Date(1710500000000);
console.log(fromTs); // → Mon Mar 18 2024 ...

// From ISO string
const fromStr = new Date("2024-03-15");
const fromFull = new Date("2024-03-15T10:30:00");

// From components (CAUTION: month is 0-indexed)
const birthday = new Date(2000, 0, 15);  // Jan 15, 2000 (0 = January)
const specific = new Date(2024, 11, 25, 8, 0, 0); // Dec 25, 2024 at 8am

// Date.now() — just the timestamp, no Date object
const ts = Date.now(); // → e.g. 1710500000000 (ms since epoch)
```

**Output**
```
new Date()          → current date/time
new Date(0)         → "Thu Jan 01 1970 00:00:00 GMT"
new Date("2024-03-15") → "Fri Mar 15 2024 00:00:00 GMT"
new Date(2000, 0, 15)  → "Sat Jan 15 2000 00:00:00 GMT"
Date.now()          → 1710500000000 (numeric timestamp)
```

---

### 2. Getter Methods — Reading Date Parts
**Theory**: Once you have a `Date` object, you can extract individual components using `get*` methods. **Critical trap**: `getMonth()` is zero-indexed (0 = January, 11 = December). Always add `+1` when displaying to users.

**Working Flow**
![flow-chart-2](flow-chart-2.png)

**Example**
```javascript
const d = new Date("2024-03-15T10:30:45");

console.log(d.getFullYear());    // → 2024
console.log(d.getMonth());       // → 2  (March — NOT 3!)
console.log(d.getMonth() + 1);   // → 3  (correct for display)
console.log(d.getDate());        // → 15
console.log(d.getDay());         // → 5  (Friday — 0=Sun)
console.log(d.getHours());       // → 10
console.log(d.getMinutes());     // → 30
console.log(d.getSeconds());     // → 45
console.log(d.getTime());        // → timestamp in ms

// Build a display string manually
const displayDate = `${d.getDate()}/${d.getMonth() + 1}/${d.getFullYear()}`;
console.log(displayDate); // → "15/3/2024"

const displayTime = `${d.getHours()}:${String(d.getMinutes()).padStart(2,'0')}`;
console.log(displayTime); // → "10:30"

// Day name
const days = ["Sun","Mon","Tue","Wed","Thu","Fri","Sat"];
console.log(days[d.getDay()]); // → "Fri"

// Month name
const months = ["Jan","Feb","Mar","Apr","May","Jun","Jul","Aug","Sep","Oct","Nov","Dec"];
console.log(months[d.getMonth()]); // → "Mar"
```

**Output**
```
d.getFullYear()  → 2024
d.getMonth()     → 2       (0-indexed! March = 2)
d.getMonth()+1   → 3       (correct human month)
d.getDate()      → 15
d.getDay()       → 5       (Friday)
d.getHours()     → 10
displayDate      → "15/3/2024"
displayTime      → "10:30"
days[d.getDay()] → "Fri"
```

---

### 3. Setter Methods — Modifying Dates
**Theory**: Each getter has a corresponding setter. Setters modify the `Date` object in place. Useful for creating dates relative to an existing one (e.g., "one month from now").

**Working Flow**
![flow-chart-3](flow-chart-3.png)

**Example**
```javascript
const d = new Date("2024-01-15");

d.setFullYear(2025);        // now 2025-01-15
d.setMonth(d.getMonth() + 3); // add 3 months → April 2025
d.setDate(d.getDate() + 10);  // add 10 days

console.log(d.toDateString()); // → "Thu Apr 24 2025" (approximately)

// Create "30 days from now"
const future = new Date();
future.setDate(future.getDate() + 30);
console.log(future.toDateString()); // → date 30 days from today

// Set to start of day
const startOfDay = new Date();
startOfDay.setHours(0, 0, 0, 0);
console.log(startOfDay); // → today at 00:00:00.000

// Set to end of day
const endOfDay = new Date();
endOfDay.setHours(23, 59, 59, 999);
```

**Output**
```
after setFullYear(2025)  → 2025-01-15
after setMonth(+3)       → 2025-04-15
after setDate(+10)       → 2025-04-25
startOfDay               → today 00:00:00.000
endOfDay                 → today 23:59:59.999
```

---

### 4. Formatting — toLocaleDateString / toLocaleTimeString
**Theory**: `toLocaleDateString()` and `toLocaleTimeString()` format dates according to locale and options. They are the cleanest built-in way to display user-friendly dates without a library.

**Working Flow**
![flow-chart-4](flow-chart-4.png)

**Example**
```javascript
const date = new Date("2024-03-15T10:30:00");

// Basic formatting by locale
console.log(date.toLocaleDateString("en-IN"));
// → "15/3/2024"

console.log(date.toLocaleDateString("en-US"));
// → "3/15/2024"

// With options
const longOptions = {
  weekday: "long",
  year:    "numeric",
  month:   "long",
  day:     "numeric"
};
console.log(date.toLocaleDateString("en-IN", longOptions));
// → "Friday, 15 March 2024"

// Time formatting
console.log(date.toLocaleTimeString("en-IN"));
// → "10:30:00 am"

console.log(date.toLocaleTimeString("en-US", { hour: "2-digit", minute: "2-digit" }));
// → "10:30 AM"

// Combined
console.log(date.toLocaleString("en-IN", {
  day: "numeric", month: "short", year: "numeric",
  hour: "2-digit", minute: "2-digit"
}));
// → "15 Mar 2024, 10:30 am"

// ISO string (good for APIs)
console.log(date.toISOString());
// → "2024-03-15T05:00:00.000Z"
```

**Output**
```
toLocaleDateString("en-IN")      → "15/3/2024"
toLocaleDateString("en-US")      → "3/15/2024"
toLocaleDateString("en-IN",long) → "Friday, 15 March 2024"
toLocaleTimeString("en-IN")      → "10:30:00 am"
toISOString()                    → "2024-03-15T05:00:00.000Z"
```

---

### 5. Date.now() and Timestamp Math
**Theory**: `Date.now()` returns the current Unix timestamp in milliseconds. Because dates are just numbers, arithmetic is straightforward: subtract two timestamps to get the time between them, add milliseconds to get a future date.

**Working Flow**
![flow-chart-5](flow-chart-5.png)

**Example**
```javascript
// Measure execution time
const start = Date.now();
let sum = 0;
for (let i = 0; i < 1_000_000; i++) sum += i;
const elapsed = Date.now() - start;
console.log(`Took ${elapsed}ms`); // → "Took 3ms"

// Time constants
const SECOND = 1000;
const MINUTE = 60 * SECOND;
const HOUR   = 60 * MINUTE;
const DAY    = 24 * HOUR;
const WEEK   = 7 * DAY;

// Future timestamps
const inOneHour = new Date(Date.now() + HOUR);
const tomorrow  = new Date(Date.now() + DAY);
const nextWeek  = new Date(Date.now() + WEEK);

console.log(tomorrow.toDateString()); // → tomorrow's date

// Days between two dates
function daysBetween(date1, date2) {
  const msPerDay = 24 * 60 * 60 * 1000;
  return Math.round(Math.abs(date2 - date1) / msPerDay);
}
const start2 = new Date("2024-01-01");
const end2   = new Date("2024-12-31");
console.log(daysBetween(start2, end2)); // → 365

// Token expiry check
function isExpired(expiryTimestamp) {
  return Date.now() > expiryTimestamp;
}
const token = { expires: Date.now() + 30 * MINUTE };
console.log(isExpired(token.expires)); // → false (just created)
```

**Output**
```
Took Xms              → e.g. "Took 3ms"
tomorrow.toDateString() → "Sat Mar 16 2024"
daysBetween(Jan1,Dec31) → 365
isExpired(token.expires)→ false
```

---

### 6. Countdown Timer Logic
**Theory**: A countdown timer subtracts the current time from a target time to get remaining milliseconds, then converts that number into days, hours, minutes, and seconds using division and modulo.

**Working Flow**
![flow-chart-6](flow-chart-6.png)

**Example**
```javascript
function getCountdown(targetDate) {
  const now       = Date.now();
  const target    = new Date(targetDate).getTime();
  const remaining = target - now;

  if (remaining <= 0) {
    return { expired: true, days: 0, hours: 0, minutes: 0, seconds: 0 };
  }

  const days    = Math.floor(remaining / (1000 * 60 * 60 * 24));
  const hours   = Math.floor((remaining % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
  const minutes = Math.floor((remaining % (1000 * 60 * 60)) / (1000 * 60));
  const seconds = Math.floor((remaining % (1000 * 60)) / 1000);

  return { expired: false, days, hours, minutes, seconds };
}

// Usage
const countdown = getCountdown("2025-01-01T00:00:00");
console.log(`${countdown.days}d ${countdown.hours}h ${countdown.minutes}m ${countdown.seconds}s`);
// → "241d 13h 29m 45s" (varies based on current time)

// Real-time countdown (browser)
function startCountdown(targetDate, onTick) {
  const interval = setInterval(() => {
    const data = getCountdown(targetDate);
    onTick(data);
    if (data.expired) clearInterval(interval);
  }, 1000);
  return interval; // return so caller can clearInterval
}

startCountdown("2025-01-01", (data) => {
  if (data.expired) {
    console.log("Happy New Year!");
  } else {
    console.log(`${data.days}d ${data.hours}h ${data.minutes}m ${data.seconds}s`);
  }
});
```

**Output**
```
getCountdown("2025-01-01") → {days:241, hours:13, minutes:29, seconds:45}
formatted                  → "241d 13h 29m 45s"
after expiry               → "Happy New Year!"
```

---

### Real-World Practical Example — Date Utilities
```javascript
// 1. Age calculator
function calculateAge(birthDateStr) {
  const birth = new Date(birthDateStr);
  const today = new Date();

  let age = today.getFullYear() - birth.getFullYear();
  const monthDiff = today.getMonth() - birth.getMonth();
  if (monthDiff < 0 || (monthDiff === 0 && today.getDate() < birth.getDate())) {
    age--; // birthday hasn't happened yet this year
  }
  return age;
}
console.log(calculateAge("2000-01-15")); // → 24 (in 2024)

// 2. Format relative time ("2 hours ago")
function timeAgo(date) {
  const seconds = Math.floor((Date.now() - new Date(date)) / 1000);
  if (seconds < 60)   return `${seconds}s ago`;
  if (seconds < 3600) return `${Math.floor(seconds / 60)}m ago`;
  if (seconds < 86400)return `${Math.floor(seconds / 3600)}h ago`;
  return `${Math.floor(seconds / 86400)}d ago`;
}
console.log(timeAgo(new Date(Date.now() - 2*60*60*1000))); // → "2h ago"
console.log(timeAgo(new Date(Date.now() - 30*1000)));      // → "30s ago"

// 3. Check if date is a weekend
function isWeekend(date = new Date()) {
  const day = new Date(date).getDay();
  return day === 0 || day === 6; // 0=Sunday, 6=Saturday
}
console.log(isWeekend("2024-03-16")); // → true  (Saturday)
console.log(isWeekend("2024-03-15")); // → false (Friday)
```

**Output**
```
calculateAge("2000-01-15")          → 24
timeAgo(2 hours ago)                → "2h ago"
timeAgo(30 seconds ago)             → "30s ago"
isWeekend("2024-03-16")             → true  (Saturday)
isWeekend("2024-03-15")             → false (Friday)
```

---

[View Interview Questions](./interview.md)
