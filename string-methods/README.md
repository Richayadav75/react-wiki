- Category: JavaScript
- Difficulty: Beginner
- Related: data-types, regex, array-methods

### JavaScript String Methods — Manipulating Text
Strings are one of the most-used data types in JavaScript. Every user input, API response, and UI label is a string. JavaScript provides rich built-in methods for searching, slicing, transforming, and formatting text.

**Key Fact**: Strings are **immutable** — no method changes the original string. Every method returns a *new* string.

**Analogy**
A string is like a printed receipt — you cannot erase a letter on the paper. But you can photocopy it and mark up the copy however you want. That copy is what the method returns.

---

### 1. Immutability — Strings Never Change
**Theory**: When you call a string method, the original string is untouched. The result is a brand new string. This is why you always need `const result = str.method()`.

**Working Flow**
![flow-chart](flow-chart.png)

**Example**
```javascript
const original = "  Hello World  ";

const trimmed  = original.trim();
const upper    = original.toUpperCase();
const replaced = original.replace("World", "JS");

console.log(original);  // → "  Hello World  "  (unchanged)
console.log(trimmed);   // → "Hello World"
console.log(upper);     // → "  HELLO WORLD  "
console.log(replaced);  // → "  Hello JS  "
```

**Output**
```
original  → "  Hello World  "   (unchanged)
trimmed   → "Hello World"
upper     → "  HELLO WORLD  "
replaced  → "  Hello JS  "
```

---

### 2. slice / substring / substr — Extracting Parts
**Theory**: Three methods for cutting out a portion of a string.
- `slice(start, end)` — supports negative indices (counts from end), does NOT swap args.
- `substring(start, end)` — no negatives (treats as 0), DOES swap if start > end.
- `substr(start, length)` — (legacy) second arg is *how many characters*, not an end index.

**Working Flow**
![flow-chart-2](flow-chart-2.png)

**Example**
```javascript
const str = "JavaScript";

// slice — negative index counts from end
console.log(str.slice(0, 4));   // → "Java"
console.log(str.slice(4));      // → "Script"  (to end)
console.log(str.slice(-6));     // → "Script"  (last 6)
console.log(str.slice(-6, -3)); // → "Scr"

// substring — no negatives
console.log(str.substring(0, 4));  // → "Java"
console.log(str.substring(4, 0));  // → "Java"  (swaps args)
console.log(str.substring(4));     // → "Script"

// substr (legacy — avoid in new code)
console.log(str.substr(4, 6));  // → "Script"
console.log(str.substr(-6, 3)); // → "Scr"  (some support negatives)
```

**Output**
```
slice(0,4)      → "Java"
slice(4)        → "Script"
slice(-6)       → "Script"
slice(-6,-3)    → "Scr"
substring(0,4)  → "Java"
substring(4,0)  → "Java"
substr(4,6)     → "Script"
```

**Explanation**: Use `slice` for modern code — it is more predictable (negative indices work, args not swapped). Avoid `substr` in new projects.

---

### 3. split / join — Converting Between String and Array
**Theory**: `split(separator)` turns a string into an array by cutting at the separator. `join(glue)` is the array method that does the reverse. Together they form a powerful transformation pipeline.

**Working Flow**
![flow-chart-3](flow-chart-3.png)

**Example**
```javascript
const csv = "alice,bob,carol,dan";

// split into array
const names = csv.split(",");
console.log(names); // → ["alice","bob","carol","dan"]
console.log(names.length); // → 4

// Split by character
"hello".split(""); // → ["h","e","l","l","o"]

// Split with limit
"a-b-c-d".split("-", 2); // → ["a","b"]  (only 2 parts)

// join back into string
names.join(", ");   // → "alice, bob, carol, dan"
names.join(" | ");  // → "alice | bob | carol | dan"
names.join("");     // → "alicebobcaroldan"

// Slug creation pipeline
const title = "The Quick Brown Fox";
const slug = title
  .toLowerCase()       // "the quick brown fox"
  .trim()              // remove extra spaces
  .split(" ")          // ["the","quick","brown","fox"]
  .join("-");          // "the-quick-brown-fox"
console.log(slug);     // → "the-quick-brown-fox"
```

**Output**
```
csv.split(",")      → ["alice","bob","carol","dan"]
names.join(", ")    → "alice, bob, carol, dan"
names.join(" | ")   → "alice | bob | carol | dan"
slug                → "the-quick-brown-fox"
```

---

### 4. includes / startsWith / endsWith / indexOf
**Theory**: Methods for searching within a string. Each answers a different question about the content.

**Working Flow**
![flow-chart-4](flow-chart-4.png)

**Example**
```javascript
const email = "alice@example.com";
const url   = "https://api.example.com/users";

// includes — case sensitive
email.includes("@");          // → true
email.includes("Alice");      // → false  (capital A)

// startsWith / endsWith
url.startsWith("https");      // → true
url.startsWith("http://");    // → false
email.endsWith(".com");       // → true
email.endsWith(".org");       // → false

// With position (optional second arg)
url.startsWith("api", 8);     // → true  (check from index 8)

// indexOf / lastIndexOf
const text = "banana";
text.indexOf("a");            // → 1  (first 'a')
text.lastIndexOf("a");        // → 5  (last 'a')
text.indexOf("z");            // → -1 (not found)

// Common pattern: check if not found
if (text.indexOf("z") === -1) {
  console.log("z not in banana"); // → "z not in banana"
}
// Modern alternative:
if (!text.includes("z")) {
  console.log("z not in banana");
}
```

**Output**
```
email.includes("@")         → true
email.includes("Alice")     → false
url.startsWith("https")     → true
email.endsWith(".com")      → true
text.indexOf("a")           → 1
text.lastIndexOf("a")       → 5
text.indexOf("z")           → -1
```

---

### 5. replace / replaceAll — Substituting Content
**Theory**: `replace(search, replacement)` replaces the **first** match. `replaceAll(search, replacement)` replaces **every** match. Both support strings and regular expressions as the search pattern.

**Working Flow**
![flow-chart-5](flow-chart-5.png)

**Example**
```javascript
const text = "The cat sat on the mat. The cat is fat.";

// replace — first match only
console.log(text.replace("cat", "dog"));
// → "The dog sat on the mat. The cat is fat."

// replaceAll — every match
console.log(text.replaceAll("cat", "dog"));
// → "The dog sat on the mat. The dog is fat."

// With regex (replace is often used with regex)
const messy = "Phone:  (123)  456-7890";
const clean = messy.replace(/\s+/g, " "); // collapse multiple spaces
console.log(clean); // → "Phone: (123) 456-7890"

// Replace with a function (dynamic replacement)
const template = "Hello {name}, your order #{orderId} is ready!";
const data = { name: "Alice", orderId: "ORD-42" };
const message = template.replace(/\{(\w+)\}/g, (_, key) => data[key] ?? `{${key}}`);
console.log(message);
// → "Hello Alice, your order #ORD-42 is ready!"

// Sanitize user input
const userInput = "<script>alert('xss')</script>";
const safe = userInput
  .replaceAll("<", "&lt;")
  .replaceAll(">", "&gt;");
console.log(safe); // → "&lt;script&gt;alert('xss')&lt;/script&gt;"
```

**Output**
```
replace("cat","dog")    → "The dog sat on the mat. The cat is fat."
replaceAll("cat","dog") → "The dog sat on the mat. The dog is fat."
clean                   → "Phone: (123) 456-7890"
message                 → "Hello Alice, your order #ORD-42 is ready!"
safe                    → "&lt;script&gt;alert('xss')&lt;/script&gt;"
```

---

### 6. trim / trimStart / trimEnd — Whitespace Cleanup
**Theory**: These methods remove whitespace (spaces, tabs, newlines) from a string. `trim()` removes from both ends. `trimStart()` (alias `trimLeft()`) removes from the start only. `trimEnd()` (alias `trimRight()`) removes from the end only.

**Example**
```javascript
const raw = "   hello world   ";

console.log(raw.trim());       // → "hello world"
console.log(raw.trimStart());  // → "hello world   "
console.log(raw.trimEnd());    // → "   hello world"

// Real-world: sanitize form input
function sanitizeInput(value) {
  return value.trim().toLowerCase().replace(/\s+/g, " ");
}
console.log(sanitizeInput("  Alice   Smith  "));
// → "alice smith"

// Tabs and newlines are also trimmed
const padded = "\t  data  \n";
console.log(padded.trim()); // → "data"
```

**Output**
```
raw.trim()       → "hello world"
raw.trimStart()  → "hello world   "
raw.trimEnd()    → "   hello world"
sanitizeInput()  → "alice smith"
padded.trim()    → "data"
```

---

### 7. padStart / padEnd / repeat — Formatting
**Theory**: `padStart(length, fill)` pads the **beginning** of a string until it reaches the target length. `padEnd` pads the end. `repeat(n)` returns the string repeated `n` times.

**Example**
```javascript
// padStart — right-align or format numbers
"5".padStart(3, "0");      // → "005"   (invoice numbers)
"42".padStart(5, " ");     // → "   42" (right-align)
"$".padStart(10, "-");     // → "---------$"

// padEnd — left-align
"Alice".padEnd(10, ".");   // → "Alice....."
"42".padEnd(5, "0");       // → "42000"

// Card masking
const cardNumber = "4111111111114242";
const masked = cardNumber.slice(-4).padStart(cardNumber.length, "*");
console.log(masked); // → "************4242"

// repeat
"ha".repeat(3);       // → "hahaha"
"-".repeat(20);       // → "--------------------" (divider line)
"0".repeat(5);        // → "00000"

// Table column formatting
function formatRow(label, value) {
  return `${label.padEnd(15)} ${String(value).padStart(8)}`;
}
console.log(formatRow("Revenue", 104500));
console.log(formatRow("Expenses", 82000));
```

**Output**
```
"5".padStart(3,"0")         → "005"
"Alice".padEnd(10,".")      → "Alice....."
masked credit card          → "************4242"
"ha".repeat(3)              → "hahaha"
formatRow("Revenue",104500) → "Revenue          104500"
formatRow("Expenses",82000) → "Expenses           82000"
```

---

### 8. toUpperCase / toLowerCase / at
**Theory**: Case conversion methods are straightforward but essential. `at(index)` is a modern alternative to bracket notation — it supports negative indices.

**Example**
```javascript
const name = "alice chen";

console.log(name.toUpperCase()); // → "ALICE CHEN"
console.log(name.toLowerCase()); // → "alice chen"

// Capitalize first letter
const capitalized = name.charAt(0).toUpperCase() + name.slice(1);
console.log(capitalized); // → "Alice chen"

// Title case
const title = "the quick brown fox";
const titleCase = title
  .split(" ")
  .map(word => word.charAt(0).toUpperCase() + word.slice(1))
  .join(" ");
console.log(titleCase); // → "The Quick Brown Fox"

// at() — negative index supported
const str = "JavaScript";
console.log(str.at(0));   // → "J"
console.log(str.at(-1));  // → "t"   (last char)
console.log(str.at(-4));  // → "r"
// str[-1] is undefined — at() is the correct modern approach
```

**Output**
```
name.toUpperCase()  → "ALICE CHEN"
capitalized         → "Alice chen"
titleCase           → "The Quick Brown Fox"
str.at(0)           → "J"
str.at(-1)          → "t"
str.at(-4)          → "r"
```

---

### Real-World Pipeline — Slug, Masking, CSV Parsing
```javascript
// 1. Create a URL slug from any title
function slugify(text) {
  return text
    .toLowerCase()
    .trim()
    .replace(/[^a-z0-9\s-]/g, "")  // remove special chars
    .replace(/\s+/g, "-")           // spaces to hyphens
    .replace(/-+/g, "-");           // collapse multiple hyphens
}
console.log(slugify("  Hello, World! ES6+ Features  "));
// → "hello-world-es6-features"

// 2. Mask sensitive data
function maskEmail(email) {
  const [user, domain] = email.split("@");
  const masked = user[0] + "*".repeat(user.length - 2) + user.at(-1);
  return `${masked}@${domain}`;
}
console.log(maskEmail("alice.smith@example.com"));
// → "a**********h@example.com"

// 3. Parse a CSV line into an object
function parseCSVLine(line, headers) {
  const values = line.split(",").map(v => v.trim());
  return Object.fromEntries(headers.map((h, i) => [h, values[i]]));
}
const headers = ["name", "age", "city"];
const row = "Alice , 28 , Mumbai";
console.log(parseCSVLine(row, headers));
// → { name:"Alice", age:"28", city:"Mumbai" }
```

**Output**
```
slugify(...)           → "hello-world-es6-features"
maskEmail(...)         → "a**********h@example.com"
parseCSVLine(...)      → {name:"Alice",age:"28",city:"Mumbai"}
```

---

[View Interview Questions](./interview.md)
