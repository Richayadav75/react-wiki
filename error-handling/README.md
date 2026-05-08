- Category: JavaScript
- Difficulty: Intermediate
- Related: promises, async-await

### Error Handling — Catching the Unexpected

Errors are inevitable. A network request can fail, a user can send invalid data, and a third-party library can throw unexpectedly. **Error handling** is how you anticipate those failures, recover gracefully, and give users helpful feedback instead of a broken page.

**Analogy**
A fire alarm system. You don't expect a fire, but you install detectors (`try`) that catch smoke (errors), trigger the alarm (`catch` — take action), and always ensure the exit doors stay unlocked (`finally` — cleanup). The building (app) keeps operating after the fire is contained.

---

### 1. try / catch / finally — The Foundation

**Theory**: Wrap risky code in `try`. If it throws, JavaScript jumps to `catch` (skipping the rest of `try`). `finally` always runs — whether an error occurred or not. Use it for cleanup: closing connections, hiding spinners, releasing locks.

**Working Flow**
![flow-chart](flow-chart.png)

**Example**
```javascript
function parseJSON(str) {
  try {
    const data = JSON.parse(str);   // may throw SyntaxError
    console.log("Parsed:", data);
    return data;
  } catch (error) {
    console.log("Parse failed:", error.message);
    return null;
  } finally {
    console.log("parseJSON finished"); // always runs
  }
}

parseJSON('{"name":"Alice"}');  // valid JSON
parseJSON("not json at all");   // invalid JSON
```

**Output**
```
Parsed: { name: 'Alice' }
parseJSON finished
Parse failed: Unexpected token 'o', "not json at all" is not valid JSON
parseJSON finished
```

**Explanation**
- On valid input: `try` succeeds, `finally` runs, returns data.
- On invalid input: `JSON.parse` throws, `catch` receives the `SyntaxError`, logs the message, returns `null`. `finally` still runs.

---

### 2. Built-in Error Types

**Theory**: JavaScript ships with specific error classes that tell you exactly what went wrong. Knowing them helps you write targeted `catch` blocks.

**Working Flow**
![flow-chart-2](flow-chart-2.png)

**Example**
```javascript
// ReferenceError
try {
  console.log(undeclaredVariable);
} catch (e) {
  console.log(e instanceof ReferenceError); // true
  console.log(e.name);    // "ReferenceError"
  console.log(e.message); // "undeclaredVariable is not defined"
}

// TypeError
try {
  null.toUpperCase();
} catch (e) {
  console.log(e instanceof TypeError); // true
  console.log(e.message); // "Cannot read properties of null"
}

// RangeError
try {
  new Array(-1);
} catch (e) {
  console.log(e instanceof RangeError); // true
  console.log(e.message); // "Invalid array length"
}
```

**Output**
```
true
ReferenceError
undeclaredVariable is not defined
true
Cannot read properties of null (reading 'toUpperCase')
true
Invalid array length
```

---

### 3. Custom Error Classes

**Theory**: In real applications, you need errors that carry domain-specific meaning — `ValidationError`, `NetworkError`, `AuthError`. Extend the built-in `Error` class to create them. This lets `catch` blocks identify the error type precisely and respond differently.

**Working Flow**
![flow-chart-3](flow-chart-3.png)

**Example**
```javascript
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = "ValidationError";
    this.field = field;
  }
}

class NetworkError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.name = "NetworkError";
    this.statusCode = statusCode;
  }
}

function validateUser(user) {
  if (!user.name) throw new ValidationError("Name is required", "name");
  if (!user.email.includes("@")) throw new ValidationError("Invalid email", "email");
  return true;
}

try {
  validateUser({ name: "", email: "notanemail" });
} catch (e) {
  if (e instanceof ValidationError) {
    console.log(`Validation failed on "${e.field}": ${e.message}`);
  } else {
    throw e; // re-throw unknown errors
  }
}
```

**Output**
```
Validation failed on "name": Name is required
```

---

### 4. Async Error Handling — try/catch with await

**Theory**: `try/catch` does NOT catch errors inside plain callbacks or raw `.then()` chains. But with `async/await`, it works exactly as expected — `await` converts rejected promises into thrown errors that `catch` can intercept.

**Working Flow**
![flow-chart-4](flow-chart-4.png)

**Example**
```javascript
// This DOES NOT work — catch never runs
try {
  setTimeout(() => {
    throw new Error("boom");
  }, 100);
} catch (e) {
  console.log("Never caught"); // this never prints
}

// This WORKS — await converts rejection to throw
async function fetchUser(id) {
  try {
    const res = await fetch(`https://api.example.com/users/${id}`);
    if (!res.ok) throw new NetworkError("Request failed", res.status);
    const data = await res.json();
    return data;
  } catch (e) {
    if (e instanceof NetworkError) {
      console.log(`Network error ${e.statusCode}: ${e.message}`);
    } else {
      console.log("Unexpected error:", e.message);
    }
    return null;
  } finally {
    console.log("fetchUser complete"); // spinner off, loading = false
  }
}
```

**Output** (when server returns 404)
```
Network error 404: Request failed
fetchUser complete
```

---

### 5. Re-throwing and Error Propagation

**Theory**: Not every `catch` should handle every error. If you catch an error you don't know how to handle, re-throw it so a higher-level handler can deal with it. This keeps each layer of code responsible for only what it understands.

**Working Flow**
![flow-chart-5](flow-chart-5.png)

**Example**
```javascript
function processForm(data) {
  try {
    validateUser(data);
    saveToDatabase(data);
  } catch (e) {
    if (e instanceof ValidationError) {
      showFieldError(e.field, e.message); // handle gracefully
    } else {
      console.error("Unexpected error in processForm:", e);
      throw e; // propagate to global handler
    }
  }
}

// Global handler for unhandled errors
window.addEventListener("error", (event) => {
  console.error("Unhandled error:", event.error);
  reportToSentry(event.error);
});

// Global handler for unhandled promise rejections
window.addEventListener("unhandledrejection", (event) => {
  console.error("Unhandled promise rejection:", event.reason);
  event.preventDefault(); // prevents default browser logging
  reportToSentry(event.reason);
});
```

---

### 6. Error Handling in Fetch — response.ok Pattern

**Theory**: `fetch()` only rejects on network failure (no connection, DNS error). A 404 or 500 HTTP response does NOT cause a rejection — it still resolves. You must check `response.ok` manually.

**Working Flow**
![flow-chart-6](flow-chart-6.png)

**Example**
```javascript
async function apiFetch(url, options = {}) {
  try {
    const response = await fetch(url, options);

    if (!response.ok) {
      const errorBody = await response.text();
      throw new NetworkError(
        `HTTP ${response.status}: ${errorBody}`,
        response.status
      );
    }

    return await response.json();
  } catch (e) {
    if (e instanceof NetworkError) {
      // Known HTTP error — handle by status code
      if (e.statusCode === 401) redirectToLogin();
      if (e.statusCode === 403) showPermissionDenied();
      if (e.statusCode >= 500) showServerError();
    } else {
      // Network failure (offline, DNS)
      showOfflineMessage();
    }
    throw e; // still propagate so callers know it failed
  }
}

// Usage
async function loadDashboard() {
  try {
    const user = await apiFetch("/api/me");
    const posts = await apiFetch("/api/posts");
    renderDashboard(user, posts);
  } catch (e) {
    console.log("Dashboard failed to load:", e.message);
  }
}
```

---

### Real-World Examples

**Form validation with custom errors**
```javascript
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = "ValidationError";
    this.field = field;
  }
}

function validateSignup({ email, password, age }) {
  if (!email.includes("@")) throw new ValidationError("Invalid email", "email");
  if (password.length < 8) throw new ValidationError("Password too short", "password");
  if (age < 18) throw new ValidationError("Must be 18+", "age");
}

async function handleSignup(formData) {
  try {
    validateSignup(formData);
    await apiFetch("/api/signup", {
      method: "POST",
      body: JSON.stringify(formData)
    });
    showSuccessMessage("Account created!");
  } catch (e) {
    if (e instanceof ValidationError) {
      highlightField(e.field, e.message);
    } else {
      showToast("Signup failed. Please try again.");
    }
  } finally {
    setLoading(false);
  }
}
```

---

[View Interview Questions](./interview.md)
