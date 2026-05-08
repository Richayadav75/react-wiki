- Category: Networking / JavaScript
- Difficulty: Intermediate
- Related: promises, async-await, api-calls-react

### Async API Calls — Fetch API Deep Dive
The Fetch API is the modern, promise-based way to make HTTP requests in the browser. It replaces the old `XMLHttpRequest`. Under the hood it uses `Request`, `Response`, and `Headers` objects — understanding these unlocks file uploads, cancellation, progress tracking, and handling every response type.

**Analogy**
Sending a courier package. You fill out a form (Request with headers and body). The courier service (browser) dispatches it. Later you receive a delivery receipt (Response). The receipt has metadata (headers: content-type, status code). You open the box to get the actual contents (response.json() / response.blob()). If you call the courier to cancel before delivery (AbortController), the package is recalled.

---

### 1. The Fetch API — Request, Response, Headers
**Theory**: `fetch()` accepts either a URL string or a `Request` object. It returns a Promise that resolves to a `Response` object. The Response body can be extracted in several formats depending on what the server sends.

**Working Flow**
![flow-chart](flow-chart.png)

**Example — All Response Types**
```javascript
// JSON response (most common)
const res  = await fetch("https://api.example.com/users/1");
const user = await res.json();         // parses JSON → JS object

// Plain text response
const textRes  = await fetch("https://api.example.com/readme");
const text     = await textRes.text();  // raw string

// Binary/file response (images, PDFs, ZIPs)
const fileRes  = await fetch("https://api.example.com/report.pdf");
const blob     = await fileRes.blob();  // Blob object
const url      = URL.createObjectURL(blob); // create local URL
// → trigger download or display in <img src={url} />

// Reading response headers
console.log(res.status);                         // 200
console.log(res.headers.get("Content-Type"));    // "application/json"
console.log(res.headers.get("X-Rate-Limit"));    // "100"
console.log(res.ok);                             // true if 200-299
```

**Output**
```
res.status                → 200
res.ok                    → true
res.headers.get(...)      → "application/json; charset=utf-8"
await res.json()          → { id: 1, name: "Richa", email: "..." }
await textRes.text()      → "# Welcome to the API\n..."
await fileRes.blob()      → Blob { size: 245760, type: "application/pdf" }
URL.createObjectURL(blob) → "blob:http://localhost:3000/abc-123-def"
```

---

### 2. POST, PUT, DELETE with Fetch
**Theory**: By default `fetch` sends a GET request. For other methods, pass an `options` object with `method`, `headers`, and `body`. The body must be a string — use `JSON.stringify()` for JSON payloads. Always set `Content-Type` header so the server knows how to parse the body.

**Working Flow**
![flow-chart-2](flow-chart-2.png)

**Example**
```javascript
const BASE = "https://jsonplaceholder.typicode.com";
const TOKEN = localStorage.getItem("token");

const headers = {
  "Content-Type": "application/json",
  "Authorization": `Bearer ${TOKEN}`,
};

// ---- CREATE (POST) ----
const createRes = await fetch(`${BASE}/posts`, {
  method: "POST",
  headers,
  body: JSON.stringify({ title: "Hello", body: "World", userId: 1 }),
});
if (!createRes.ok) throw new Error(`Create failed: ${createRes.status}`);
const newPost = await createRes.json();
// → { id: 101, title: "Hello", body: "World", userId: 1 }

// ---- UPDATE (PUT) ----
const updateRes = await fetch(`${BASE}/posts/1`, {
  method: "PUT",
  headers,
  body: JSON.stringify({ id: 1, title: "Updated Title", body: "Updated body", userId: 1 }),
});
const updated = await updateRes.json();
// → { id: 1, title: "Updated Title", ... }

// ---- PARTIAL UPDATE (PATCH) ----
const patchRes = await fetch(`${BASE}/posts/1`, {
  method: "PATCH",
  headers,
  body: JSON.stringify({ title: "Just the title changed" }),
});
const patched = await patchRes.json();

// ---- DELETE ----
const deleteRes = await fetch(`${BASE}/posts/1`, {
  method: "DELETE",
  headers,
});
// DELETE often returns 200 with empty body or 204 No Content
console.log(deleteRes.status); // 200
```

**Output**
```
POST  /posts         → 201  { id: 101, title: "Hello", body: "World", userId: 1 }
PUT   /posts/1       → 200  { id: 1, title: "Updated Title", body: "Updated body" }
PATCH /posts/1       → 200  { id: 1, title: "Just the title changed", body: "..." }
DELETE /posts/1      → 200  {}
```

---

### 3. AbortController — Cancelling Requests
**Theory**: `AbortController` lets you programmatically cancel in-flight fetch requests. The controller emits an abort signal. Pass this signal to fetch — when you call `controller.abort()`, the fetch is cancelled and throws an `AbortError`. This is essential for: cleanup in `useEffect`, search-as-you-type, and navigation away from a page.

**Working Flow**
![flow-chart-3](flow-chart-3.png)

**Example — Cancellable Search**
```javascript
let searchController = null;

async function searchUsers(query) {
  // Cancel any previous search request
  if (searchController) {
    searchController.abort();
  }

  searchController = new AbortController();

  try {
    const res  = await fetch(`/api/users?q=${encodeURIComponent(query)}`, {
      signal: searchController.signal,
    });
    const data = await res.json();
    renderResults(data);
  } catch (err) {
    if (err.name === "AbortError") {
      console.log("Search cancelled — newer query in flight");
      return;  // intentional, not an error
    }
    console.error("Search failed:", err);
  }
}

// In React — cancellable search input
function SearchBox() {
  const [query, setQuery]     = React.useState("");
  const [results, setResults] = React.useState([]);

  React.useEffect(() => {
    if (!query) return;
    const controller = new AbortController();

    fetch(`/api/users?q=${query}`, { signal: controller.signal })
      .then(r => r.json())
      .then(setResults)
      .catch(err => { if (err.name !== "AbortError") console.error(err); });

    return () => controller.abort(); // cancel when query changes or component unmounts
  }, [query]);

  return (
    <>
      <input value={query} onChange={e => setQuery(e.target.value)} placeholder="Search..." />
      <ul>{results.map(u => <li key={u.id}>{u.name}</li>)}</ul>
    </>
  );
}
```

**Output**
```
User types "R"       → fetch("/api/users?q=R") starts [A]
User types "Ri"      → [A] aborted, fetch("/api/users?q=Ri") starts [B]
User types "Ric"     → [B] aborted, fetch("/api/users?q=Ric") starts [C]
[C] resolves         → results = [{ name: "Richa" }]   ← correct, no stale data
```

---

### 4. Retry Logic
**Theory**: Network requests can fail transiently (server hiccup, timeout). Retry logic re-attempts the request a set number of times with exponential backoff (wait longer between each retry) before giving up.

**Working Flow**
![flow-chart-4](flow-chart-4.png)

**Example**
```javascript
async function fetchWithRetry(url, options = {}, retries = 3, delay = 500) {
  for (let attempt = 1; attempt <= retries; attempt++) {
    try {
      const res = await fetch(url, options);
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      return await res.json();
    } catch (err) {
      if (err.name === "AbortError") throw err; // don't retry cancellations
      if (attempt === retries) throw err;       // final attempt — propagate error
      console.warn(`Attempt ${attempt} failed. Retrying in ${delay}ms...`);
      await new Promise(resolve => setTimeout(resolve, delay));
      delay *= 2; // exponential backoff: 500 → 1000 → 2000
    }
  }
}

// Usage
const data = await fetchWithRetry("/api/posts", {}, 3, 500);
```

**Output**
```
Attempt 1 → 503 Service Unavailable → wait 500ms
Attempt 2 → 503 Service Unavailable → wait 1000ms
Attempt 3 → 200 OK                  → return data
───
Attempt 1 → Network Error
Attempt 2 → Network Error
Attempt 3 → Network Error → throws Error("HTTP 500") to caller
```

---

### 5. Parallel API Calls with `Promise.all`
**Theory**: When you need data from multiple independent endpoints, fire all requests simultaneously with `Promise.all`. This takes an array of Promises and resolves when ALL resolve, or rejects immediately if ANY one fails. Compare to sequential: 3 requests × 300ms = 900ms. Parallel: max(300,200,250)ms = 300ms.

**Working Flow**
![flow-chart-5](flow-chart-5.png)

**Example**
```javascript
// Sequential — slow
const user    = await fetch("/api/users/1").then(r => r.json());  // 300ms
const posts   = await fetch("/api/posts?userId=1").then(r => r.json()); // 200ms
const friends = await fetch("/api/friends/1").then(r => r.json()); // 250ms
// Total: ~750ms

// Parallel — fast
const [user, posts, friends] = await Promise.all([
  fetch("/api/users/1").then(r => r.json()),
  fetch("/api/posts?userId=1").then(r => r.json()),
  fetch("/api/friends/1").then(r => r.json()),
]);
// Total: ~300ms (limited by slowest)

// Promise.allSettled — don't fail if one request fails
const results = await Promise.allSettled([
  fetch("/api/users/1").then(r => r.json()),
  fetch("/api/broken-endpoint").then(r => r.json()),  // will fail
  fetch("/api/posts").then(r => r.json()),
]);

results.forEach(result => {
  if (result.status === "fulfilled") {
    console.log("Got data:", result.value);
  } else {
    console.warn("Request failed:", result.reason.message);
  }
});
```

**Output**
```
Promise.all (all succeed):
  [user, posts, friends] = [{ id:1, name:"Richa" }, [{...},...], [{...},...]]
  Time: ~300ms

Promise.all (one fails):
  → rejects immediately with that error — other requests are abandoned

Promise.allSettled:
  results[0] → { status: "fulfilled", value: { id:1, name:"Richa" } }
  results[1] → { status: "rejected",  reason: Error("HTTP 404") }
  results[2] → { status: "fulfilled", value: [...posts] }
  → partial data displayed, broken endpoint gracefully skipped
```

---

### 6. File Upload with Progress Tracking
**Theory**: Uploading a file with `fetch` uses `FormData` as the body. For progress tracking, you must use `XMLHttpRequest` because Fetch API does not expose upload progress. A common pattern wraps XHR in a Promise.

**Working Flow**
![flow-chart-6](flow-chart-6.png)

**Example**
```jsx
function FileUploader() {
  const [progress, setProgress] = React.useState(0);
  const [status, setStatus]     = React.useState("idle"); // idle | uploading | done | error

  function uploadFile(file) {
    return new Promise((resolve, reject) => {
      const formData = new FormData();
      formData.append("file", file);
      formData.append("userId", "42");

      const xhr = new XMLHttpRequest();

      // Track upload progress
      xhr.upload.onprogress = (event) => {
        if (event.lengthComputable) {
          const percent = Math.round((event.loaded / event.total) * 100);
          setProgress(percent);
        }
      };

      xhr.onload = () => {
        if (xhr.status >= 200 && xhr.status < 300) {
          resolve(JSON.parse(xhr.responseText));
        } else {
          reject(new Error(`Upload failed: ${xhr.status}`));
        }
      };

      xhr.onerror = () => reject(new Error("Network error"));

      xhr.open("POST", "/api/upload");
      xhr.setRequestHeader("Authorization", `Bearer ${localStorage.getItem("token")}`);
      xhr.send(formData);
    });
  }

  async function handleChange(e) {
    const file = e.target.files[0];
    if (!file) return;
    setStatus("uploading");
    setProgress(0);
    try {
      const result = await uploadFile(file);
      setStatus("done");
      console.log("Uploaded:", result.url);
    } catch (err) {
      setStatus("error");
      console.error(err.message);
    }
  }

  return (
    <div>
      <input type="file" onChange={handleChange} disabled={status === "uploading"} />
      {status === "uploading" && (
        <div>
          <progress value={progress} max="100" />
          <span>{progress}%</span>
        </div>
      )}
      {status === "done"  && <p>Upload complete!</p>}
      {status === "error" && <p>Upload failed. Please try again.</p>}
    </div>
  );
}
```

**Output**
```
(initial)    → [Choose File] button
(uploading)  → [==========>    ] 65%
(complete)   → "Upload complete!"
(error)      → "Upload failed. Please try again."
```

---

### Real-World Example: Cancellable Search + File Download
```javascript
// --- Cancellable search ---
const controller = new AbortController();

const results = await fetch(`/api/search?q=${query}`, {
  signal: controller.signal,
}).then(r => r.json());

// Cancel it:
controller.abort(); // → AbortError thrown, caught gracefully

// --- File download with blob ---
async function downloadReport(reportId) {
  const res  = await fetch(`/api/reports/${reportId}`, {
    headers: { Authorization: `Bearer ${token}` },
  });

  if (!res.ok) throw new Error("Download failed");

  const blob     = await res.blob();
  const blobUrl  = URL.createObjectURL(blob);

  // Trigger browser download
  const link     = document.createElement("a");
  link.href      = blobUrl;
  link.download  = `report-${reportId}.pdf`;
  link.click();

  // Clean up blob URL to free memory
  URL.revokeObjectURL(blobUrl);
}
```

**Output**
```
downloadReport("Q1-2025")
→ fetch /api/reports/Q1-2025
→ res.blob() → Blob { size: 2097152, type: "application/pdf" }
→ browser shows download dialog: "report-Q1-2025.pdf"
→ URL revoked — memory freed
```

---

[View Interview Questions](./interview.md)
