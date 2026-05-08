- Category: JavaScript
- Difficulty: Intermediate
- Related: string-methods, es6-features

### Regular Expressions — Pattern Matching in Strings
A Regular Expression (regex) is a sequence of characters that forms a search pattern. It can be used to check if a string contains a pattern, find all matches, replace text, or split a string at specific points. Every major programming language supports regex — it is a universal tool for working with text.

**Analogy**
A metal stencil for painting. You hold it over a canvas (string) and spray paint through it — only the shapes cut into the stencil (pattern) transfer to the canvas. The stencil works the same way on any canvas, and you can spray in different colours (flags like `g`, `i`).

---

### 1. Creating a Regex — Literal vs Constructor

**Theory**
Two ways to create a regex:
- **Literal** `/pattern/flags` — used when the pattern is known at write time (most common)
- **Constructor** `new RegExp("pattern", "flags")` — used when the pattern is dynamic (built from variables)

**Working Flow**

![flow-chart](flow-chart.png)

**Example**
```javascript
// Literal — write the pattern between slashes
const literalRegex = /hello/i;   // i = case insensitive

// Constructor — pattern is a string (escape backslashes!)
const searchTerm = 'hello';
const dynRegex   = new RegExp(searchTerm, 'i');

// Both behave identically
literalRegex.test('Hello World');  // → true
dynRegex.test('Hello World');      // → true

// When to use constructor — user-supplied search term
function searchInText(text, term) {
  const pattern = new RegExp(term, 'gi'); // gi = global + case-insensitive
  return text.match(pattern);
}

searchInText('The cat sat on the mat', 'at');
// → ["at", "at", "at"]
```

**Output**
```
literalRegex.test("Hello World")   → true
dynRegex.test("Hello World")       → true
searchInText("The cat...", "at")   → ["at", "at", "at"]
```

---

### 2. Character Classes — Matching Types of Characters

**Theory**
Character classes let you match any character from a set, rather than one specific character.

| Class | Matches | Example |
| :--- | :--- | :--- |
| `[abc]` | a, b, or c | `[aeiou]` matches vowels |
| `[a-z]` | any lowercase letter | `[a-zA-Z]` any letter |
| `[0-9]` | any digit | same as `\d` |
| `\d` | digit (0-9) | `\d\d\d` matches "123" |
| `\w` | word char (a-z, A-Z, 0-9, _) | `\w+` matches a word |
| `\s` | whitespace (space, tab, newline) | `\s+` matches gaps |
| `\D` | NOT a digit | `\D+` matches "abc" |
| `.` | any character except newline | `a.b` matches "aXb" |

**Working Flow**

![flow-chart-2](flow-chart-2.png)

**Example**
```javascript
const text = 'My phone is 98765-43210 and PIN is 4821';

// \d+ — one or more digits
text.match(/\d+/g);
// → ["98765", "43210", "4821"]

// [aeiou] — any vowel
'Hello World'.match(/[aeiou]/gi);
// → ["e", "o", "o"]

// \w+ — words
'Hello, World! 123'.match(/\w+/g);
// → ["Hello", "World", "123"]

// \s — whitespace
'one two  three'.split(/\s+/);
// → ["one", "two", "three"]

// [^0-9] — NOT a digit (^ inside [] = negation)
'abc123def'.replace(/[^0-9]/g, '');
// → "123"
```

**Output**
```
\d+    match      → ["98765", "43210", "4821"]
[aeiou] match     → ["e", "o", "o"]
\w+ match         → ["Hello", "World", "123"]
split on \s+      → ["one", "two", "three"]
remove non-digits → "123"
```

---

### 3. Quantifiers — How Many Times to Match

**Theory**
Quantifiers tell the regex engine how many times a character or group can appear.

| Quantifier | Meaning | Example |
| :--- | :--- | :--- |
| `*` | 0 or more | `ab*c` → "ac", "abc", "abbc" |
| `+` | 1 or more | `ab+c` → "abc", "abbc" (not "ac") |
| `?` | 0 or 1 (optional) | `colou?r` → "color" or "colour" |
| `{n}` | exactly n | `\d{4}` → exactly 4 digits |
| `{n,}` | n or more | `\d{3,}` → 3+ digits |
| `{n,m}` | between n and m | `\d{2,4}` → 2, 3, or 4 digits |

**Working Flow**

![flow-chart-3](flow-chart-3.png)

**Example**
```javascript
// {4} — exactly 4 digits (year)
'2024-05-15'.match(/\d{4}/);
// → ["2024"]

// {2,4} — 2 to 4 word characters
'Hi Hello'.match(/\w{2,4}/g);
// → ["Hi", "Hell"]  (Hello has 5 chars, matches first 4)

// ? — optional character (British/American spelling)
'The colour and the color'.match(/colou?r/g);
// → ["colour", "color"]

// + vs * — must have at least one digit vs can have zero
'abc'.match(/\d+/);   // → null  (no digits)
'abc'.match(/\d*/);   // → [""]  (zero digits matches empty string)
```

---

### 4. Anchors and Groups

**Theory**
- **Anchors** pin a match to a position in the string (not a character)
- **Groups** `()` capture a part of the match or group alternatives

| Symbol | Meaning |
| :--- | :--- |
| `^` | Start of string (or line with `m` flag) |
| `$` | End of string |
| `\b` | Word boundary |
| `(abc)` | Capturing group — captured in result |
| `(?:abc)` | Non-capturing group — groups without capturing |
| `a\|b` | Alternation — a or b |

**Working Flow**

![flow-chart-4](flow-chart-4.png)

**Example**
```javascript
// ^ and $ — must match the ENTIRE string
/^\d{10}$/.test('9876543210');    // → true   (exactly 10 digits)
/^\d{10}$/.test('98765432101');   // → false  (11 digits)
/^\d{10}$/.test('987654321a');    // → false  (non-digit at end)

// \b — whole word match
'cat concatenate'.match(/\bcat\b/g);
// → ["cat"]  (only the standalone word)

// () — capturing groups
const date = '2024-05-15';
const m    = date.match(/(\d{4})-(\d{2})-(\d{2})/);
// m[0] = "2024-05-15"  (full match)
// m[1] = "2024"        (group 1)
// m[2] = "05"          (group 2)
// m[3] = "15"          (group 3)

// | — alternation
/cat|dog/.test('I have a dog');   // → true
/^(Mr|Mrs|Ms)\./.test('Ms. Smith'); // → true
```

**Output**
```
/^\d{10}$/.test("9876543210")  → true
\bcat\b in "cat concatenate"   → ["cat"]
date groups: year=2024, month=05, day=15
/cat|dog/.test("I have a dog") → true
```

---

### 5. Flags and String Methods

**Theory**
Flags modify how the regex behaves. They go after the closing slash: `/pattern/flags`.

| Flag | Name | Effect |
| :--- | :--- | :--- |
| `g` | Global | Find all matches, not just first |
| `i` | Insensitive | Case-insensitive matching |
| `m` | Multiline | `^`/`$` match start/end of each line |
| `s` | Dotall | `.` matches newlines too |

**Working Flow**

![flow-chart-5](flow-chart-5.png)

**Example**
```javascript
const text = 'Hello hello HELLO';

// Without g — only first match
text.match(/hello/i);        // → ["Hello"]

// With g — all matches
text.match(/hello/gi);       // → ["Hello", "hello", "HELLO"]

// test — returns boolean
/^\d+$/.test('12345');       // → true  (all digits)
/^\d+$/.test('123ab');       // → false

// replace — with g replaces all
'aaa bbb aaa'.replace(/aaa/g, 'XXX');  // → "XXX bbb XXX"

// matchAll — returns iterator with all capture groups
const emails = 'user@a.com, admin@b.org';
const regex  = /(\w+)@(\w+)\.(\w+)/g;
const matches = [...emails.matchAll(regex)];
// matches[0] = ["user@a.com",  "user",  "a",   "com"]
// matches[1] = ["admin@b.org", "admin", "b",   "org"]

// split with regex
'one1two2three3four'.split(/\d/);
// → ["one", "two", "three", "four"]
```

---

### 6. Real-World — Common Validation Patterns

**Working Flow**

![flow-chart-6](flow-chart-6.png)

**Example**
```javascript
// Email validation
const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]{2,}$/;
emailRegex.test('user@example.com');   // → true
emailRegex.test('user@.com');          // → false
emailRegex.test('user example.com');   // → false

// Password strength (min 8 chars, 1 uppercase, 1 number, 1 special)
const strongPassword = /^(?=.*[A-Z])(?=.*\d)(?=.*[!@#$%]).{8,}$/;
strongPassword.test('Weak1!pass');     // → true
strongPassword.test('weakpassword');   // → false

// Indian phone number (10 digits, optionally starting with +91)
const phoneRegex = /^(\+91[\s-]?)?[6-9]\d{9}$/;
phoneRegex.test('9876543210');         // → true
phoneRegex.test('+91 9876543210');     // → true

// URL slug (lowercase, hyphens, no spaces)
function toSlug(text) {
  return text
    .toLowerCase()
    .trim()
    .replace(/[^\w\s-]/g, '')   // remove non-word chars except hyphens
    .replace(/\s+/g, '-')       // spaces → hyphens
    .replace(/-+/g, '-');       // multiple hyphens → one
}
toSlug('Hello World!! How Are You?');
// → "hello-world-how-are-you"

// Extract all hashtags from a tweet
const tweet = 'Loving #React and #JavaScript today! #coding';
tweet.match(/#\w+/g);
// → ["#React", "#JavaScript", "#coding"]
```

**Output**
```
emailRegex.test("user@example.com")  → true
strongPassword.test("Weak1!pass")    → true
phoneRegex.test("9876543210")        → true
toSlug("Hello World!! How Are You?") → "hello-world-how-are-you"
hashtags in tweet                    → ["#React", "#JavaScript", "#coding"]
```

---

[View Interview Questions](./interview.md)
