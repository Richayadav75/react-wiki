# String Methods — Interview Questions

---

**1. Are strings mutable in JavaScript? Why does this matter?**

No — strings are **immutable**. No method can change the original string. Every string method returns a brand-new string. This matters because you must always capture the result:

```javascript
const name = "alice";
name.toUpperCase(); // returns "ALICE" but name is still "alice"!

const upper = name.toUpperCase(); // correct — save the result
console.log(upper); // → "ALICE"
console.log(name);  // → "alice" (unchanged)
```

---

**2. What is the difference between `slice()` and `substring()`?**

| Behavior | `slice()` | `substring()` |
|----------|-----------|---------------|
| Negative indices | Counts from end | Treated as `0` |
| Start > End | Returns `""` | Swaps the arguments |

```javascript
const str = "JavaScript";

str.slice(0, 4);       // → "Java"
str.slice(-6);         // → "Script"   (last 6 from end)
str.slice(8, 4);       // → ""         (start > end)

str.substring(0, 4);   // → "Java"
str.substring(-6);     // → "JavaScript" (-6 treated as 0)
str.substring(8, 4);   // → "Scri"    (swaps to substring(4,8))
```

Prefer `slice` — negative indices and predictable behavior make it more versatile.

---

**3. How do you create a URL-friendly slug from a string?**

```javascript
function slugify(text) {
  return text
    .toLowerCase()
    .trim()
    .replace(/[^a-z0-9\s-]/g, "")  // remove special chars
    .replace(/\s+/g, "-")           // spaces → hyphens
    .replace(/-+/g, "-");           // collapse multiple hyphens
}

console.log(slugify("  Hello, World! ES6+ Features  "));
// → "hello-world-es6-features"
```

---

**4. How do you replace all occurrences of a substring?**

Two approaches:

```javascript
const text = "cat and cat and cat";

// Option 1 — replaceAll (ES2021, clean)
text.replaceAll("cat", "dog");
// → "dog and dog and dog"

// Option 2 — replace with global regex (older but always works)
text.replace(/cat/g, "dog");
// → "dog and dog and dog"

// Note: replace("cat", "dog") only replaces the FIRST match
text.replace("cat", "dog");
// → "dog and cat and cat"
```

---

**5. How do you check if a string starts with, ends with, or contains a value?**

```javascript
const url = "https://api.example.com/users";

url.startsWith("https");    // → true
url.startsWith("http://");  // → false
url.endsWith("/users");     // → true
url.includes("example");    // → true
url.includes("Example");    // → false (case-sensitive)

// With position argument
url.startsWith("api", 8);   // → true (check starting at index 8)

// indexOf returns position (or -1 if not found)
url.indexOf("example");     // → 12
url.indexOf("missing");     // → -1
```

---

**6. Explain `padStart()` and `padEnd()`. Give a real use case.**

Both pad a string to a target length. `padStart` adds characters at the beginning, `padEnd` at the end.

```javascript
// Invoice / order number formatting
"42".padStart(6, "0");     // → "000042"
"99".padStart(6, "0");     // → "000099"

// Credit card masking
const card = "4111111111114242";
const masked = card.slice(-4).padStart(card.length, "*");
console.log(masked); // → "************4242"

// Table alignment
"Revenue".padEnd(15, " ")  + "104,500".padStart(10);
// → "Revenue         104,500"
```

---

**7. What does `split()` return when the separator is not found?**

It returns an array with the entire string as the only element:

```javascript
"hello".split(",");  // → ["hello"]  (one element, no split happened)
"hello".split("");   // → ["h","e","l","l","o"]  (split every char)
"a-b-c".split("-", 2); // → ["a","b"]  (limit to 2 parts)
```

---

**8. What is `trim()` and what kinds of whitespace does it remove?**

`trim()` removes leading and trailing whitespace — spaces, tabs (`\t`), newlines (`\n`), carriage returns (`\r`).

```javascript
"  hello  ".trim();       // → "hello"
"\t\n  data  \n\t".trim();// → "data"

// trimStart / trimEnd for one side only
"  hi  ".trimStart();     // → "hi  "
"  hi  ".trimEnd();       // → "  hi"

// Real use: sanitizing form input
const input = "  Alice Smith  ";
const clean = input.trim().replace(/\s+/g, " ");
// → "Alice Smith"
```

---

**9. How do you convert between a string and an array?**

`split()` converts string → array. The array's `join()` method converts array → string.

```javascript
// String to array
const csv = "red,green,blue";
const colors = csv.split(","); // → ["red","green","blue"]

// Array to string
colors.join(" | ");   // → "red | green | blue"
colors.join("");      // → "redgreenblue"

// Powerful pipeline
"The Quick Brown Fox"
  .toLowerCase()      // "the quick brown fox"
  .split(" ")         // ["the","quick","brown","fox"]
  .map(w => w[0].toUpperCase() + w.slice(1))  // capitalize
  .join(" ");         // "The Quick Brown Fox"
```

---

**10. What does the `at()` method do differently from bracket notation?**

`at(index)` supports **negative indices**, counting from the end. Bracket notation `str[-1]` returns `undefined` — it looks for a key named `"-1"`.

```javascript
const str = "JavaScript";

str[0];      // → "J"
str[-1];     // → undefined  (no key "-1")
str[str.length - 1]; // → "t"  (old way for last char)

str.at(0);   // → "J"
str.at(-1);  // → "t"   (modern, clean)
str.at(-4);  // → "r"

// at() also works on arrays
[10, 20, 30].at(-1); // → 30
```
