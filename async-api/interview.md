# Async API Calls — Interview Questions

---

**1. What is the Fetch API and how does it differ from XMLHttpRequest?**

`fetch` is the modern, Promise-based browser API for HTTP requests. `XMLHttpRequest` (XHR) is the older callback-based API.

```javascript
// XHR — verbose, callback-based
const xhr = new XMLHttpRequest();
xhr.open("GET", "/api/users");
xhr.onload  = () => console.log(JSON.parse(xhr.responseText));
xhr.onerror = () => console.error("Failed");
xhr.send();

// fetch — clean, Promise-based, async/await friendly
const data = await fetch("/api/users").then(r => r.json());
```

Key differences:
- `fetch` returns a Promise; XHR uses callbacks
- `fetch` does NOT reject on HTTP errors (4xx/5xx) — you must check `res.ok`
- XHR supports upload progress; `fetch` does not (use XHR for progress bars)
- `fetch` supports `AbortController`; XHR uses `xhr.abort()`

---

**2. Why does `fetch` not throw on 4xx/5xx errors and how do you handle it?**

`fetch` only rejects on network failures (DNS failure, no internet). A 404 or 500 response is still a "successful" HTTP communication, so `fetch` resolves normally.

```javascript
// WRONG — silently treats 404 as success
const data = await fetch("/api/users/999").then(r => r.json());

// CORRECT — always check res.ok
const res  = await fetch("/api/users/999");
if (!res.ok) {
  throw new Error(`HTTP ${res.status}: ${res.statusText}`);
}
const data = await res.json();
```

```text
fetch("/broken")  →  res.ok = false, res.status = 404
                  →  res.json() still works (parses error body)
                  →  you must throw manually
```

---

**3. What are the different ways to read a Response body?**

The `Response` object has several methods to extract different data formats. Each returns a Promise and can only be called once (body is a stream).

```javascript
const res = await fetch("/api/endpoint");

// JSON data (objects, arrays)
const json = await res.json();

// Plain text (HTML, CSV, plain text files)
const text = await res.text();

// Binary data (images, PDFs, audio, zip files)
const blob = await res.blob();

// ArrayBuffer (low-level binary, for WebGL, audio processing)
const buffer = await res.arrayBuffer();

// FormData (for multipart responses)
const form = await res.formData();

// Warning — body can only be consumed once!
const res2 = await fetch("/api/data");
await res2.json();           // ✓ works
await res2.json();           // ✗ TypeError: body already used
```

---

**4. How does `AbortController` work and what are its use cases?**

`AbortController` creates a controller object with a `signal`. Pass the signal to `fetch`. Calling `controller.abort()` sends a signal that cancels the in-flight request.

```javascript
const controller = new AbortController();

// Start request with signal attached
fetch("/api/data", { signal: controller.signal })
  .then(r => r.json())
  .catch(err => {
    if (err.name === "AbortError") {
      console.log("Request intentionally cancelled");
    }
  });

// Cancel it 2 seconds later
setTimeout(() => controller.abort(), 2000);
```

Use cases:
1. **React `useEffect` cleanup** — cancel fetch when component unmounts
2. **Race condition fix** — cancel previous request when dependency changes
3. **Search-as-you-type** — cancel search when user types another character
4. **Timeout** — abort if server doesn't respond within N seconds

```javascript
// Timeout pattern using AbortController
const controller = new AbortController();
const timeout    = setTimeout(() => controller.abort(), 5000); // 5s timeout

try {
  const res  = await fetch("/api/slow", { signal: controller.signal });
  const data = await res.json();
  clearTimeout(timeout);
  return data;
} catch (err) {
  if (err.name === "AbortError") throw new Error("Request timed out");
  throw err;
}
```

---

**5. What is the difference between `Promise.all` and `Promise.allSettled`?**

```javascript
// Promise.all — fails fast: if ANY promise rejects, all results are lost
const [users, posts] = await Promise.all([
  fetch("/api/users").then(r => r.json()),
  fetch("/api/posts").then(r => r.json()),  // if this throws → everything throws
]);

// Promise.allSettled — waits for ALL, reports success/failure per promise
const results = await Promise.allSettled([
  fetch("/api/users").then(r => r.json()),
  fetch("/api/broken").then(r => r.json()),
]);

results.forEach(result => {
  if (result.status === "fulfilled") use(result.value);
  else console.error("Failed:", result.reason);
});
```

```text
Promise.all:        if 1 of 3 fails → all 3 lost, catch block runs
Promise.allSettled: if 1 of 3 fails → you get 2 values + 1 error, no throw
```

Use `Promise.all` when all requests are critical. Use `Promise.allSettled` for dashboard panels where each widget is independent.

---

**6. How do you implement retry logic with exponential backoff?**

```javascript
async function fetchWithRetry(url, maxRetries = 3, baseDelay = 500) {
  let lastError;

  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const res = await fetch(url);
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      return await res.json(); // success — return immediately
    } catch (err) {
      if (err.name === "AbortError") throw err; // don't retry cancellations
      lastError = err;
      if (attempt < maxRetries) {
        const wait = baseDelay * Math.pow(2, attempt - 1); // 500, 1000, 2000
        console.warn(`Attempt ${attempt} failed. Waiting ${wait}ms...`);
        await new Promise(r => setTimeout(r, wait));
      }
    }
  }

  throw lastError; // all retries exhausted
}
```

```text
Attempt 1 → fail → 500ms
Attempt 2 → fail → 1000ms
Attempt 3 → fail → throw Error to caller

Attempt 1 → fail → 500ms
Attempt 2 → success → return data immediately
```

---

**7. What is the interceptor pattern and how would you implement it with fetch?**

Interceptors are middleware functions that run before every request or after every response. Axios has built-in interceptors; with `fetch` you implement them by wrapping.

```javascript
// Request interceptor — add auth token to every request
async function apiFetch(url, options = {}) {
  // Before request: inject headers
  const token = localStorage.getItem("token");
  const mergedOptions = {
    ...options,
    headers: {
      "Content-Type": "application/json",
      ...(token ? { Authorization: `Bearer ${token}` } : {}),
      ...options.headers,
    },
  };

  const res = await fetch(url, mergedOptions);

  // After response: handle 401 globally
  if (res.status === 401) {
    localStorage.removeItem("token");
    window.location.href = "/login"; // redirect to login
    return;
  }

  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
}

// Usage — no need to add auth headers manually anywhere
const user = await apiFetch("/api/profile");
const posts = await apiFetch("/api/posts", { method: "POST", body: JSON.stringify(data) });
```

---

**8. How do you handle file downloads using the Fetch API?**

```javascript
async function downloadFile(url, filename) {
  const res  = await fetch(url, {
    headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
  });

  if (!res.ok) throw new Error("Download failed");

  // Get binary data as Blob
  const blob    = await res.blob();

  // Create a temporary object URL
  const blobUrl = URL.createObjectURL(blob);

  // Trigger browser download dialog
  const link    = document.createElement("a");
  link.href     = blobUrl;
  link.download = filename;
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);

  // Free memory — important!
  URL.revokeObjectURL(blobUrl);
}

// Usage
await downloadFile("/api/reports/Q1-2025.pdf", "Q1-2025-report.pdf");
```

---

**9. What is the difference between `response.json()` and `JSON.parse(response.text())`?**

They produce the same result, but `response.json()` is the idiomatic way.

```javascript
// Both are equivalent:
const data1 = await res.json();
const data2 = JSON.parse(await res.text());

// Why prefer res.json():
// 1. Cleaner syntax
// 2. Can only read the body once — res.json() reads AND parses in one step
// 3. res.text() reads the body, leaving nothing for res.json() to read

// When to use res.text():
const text = await res.text(); // use for CSV, HTML, plain text, or to inspect raw body
```

---

**10. How do you upload a file with progress tracking?**

`fetch` does not expose upload progress events. Use `XMLHttpRequest` wrapped in a Promise for progress, or use `fetch` for simplicity without progress.

```javascript
function uploadWithProgress(file, onProgress) {
  return new Promise((resolve, reject) => {
    const formData = new FormData();
    formData.append("file", file);

    const xhr = new XMLHttpRequest();

    // Progress events — only available via XHR
    xhr.upload.addEventListener("progress", (e) => {
      if (e.lengthComputable) {
        onProgress(Math.round((e.loaded / e.total) * 100));
      }
    });

    xhr.addEventListener("load", () => {
      if (xhr.status >= 200 && xhr.status < 300) {
        resolve(JSON.parse(xhr.responseText));
      } else {
        reject(new Error(`Upload failed: ${xhr.status}`));
      }
    });

    xhr.addEventListener("error", () => reject(new Error("Network error")));

    xhr.open("POST", "/api/upload");
    xhr.send(formData);
  });
}

// React usage
const result = await uploadWithProgress(file, (percent) => {
  setProgress(percent); // 0 → 25 → 67 → 100
});
```

```text
0%   → progress bar empty
25%  → [====                ]
67%  → [=============       ]
100% → [====================] Upload complete!
```
