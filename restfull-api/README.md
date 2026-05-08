- Category: Web Development
- Difficulty: Beginner
- Related: json, async-await, api-calls-react

### RESTful API — How Clients and Servers Talk

A **RESTful API** is a convention for designing HTTP-based communication between a client (browser/app) and a server. REST stands for Representational State Transfer. It defines rules — not code — for how resources should be structured, accessed, and modified using standard HTTP.

**Analogy**
A restaurant menu system. The client (customer) asks for resources (dishes) using standard verbs: "GET me the menu", "POST an order", "PUT a different order", "DELETE my booking". The kitchen (server) processes requests and sends responses (food/receipts). The waiter (HTTP) is the middleman who carries messages back and forth using a standard format everyone understands.

---

### 1. REST Principles

**Theory**: REST is an architectural style, not a protocol. APIs following these 6 constraints are called RESTful.

**Working Flow**
![flow-chart](flow-chart.png)

**Most important in practice**: Stateless + Uniform Interface

**Uniform Interface means:**
```
Resource = a noun (thing)       /users, /posts, /orders/42
Verb = what you do to it        GET, POST, PUT, PATCH, DELETE
Representation = the data sent  JSON (almost always)
```

---

### 2. HTTP Methods — CRUD Mapping

**Theory**: REST maps the four CRUD operations to HTTP methods. Each method has a semantic meaning — it tells the server what operation to perform.

**Working Flow**
![flow-chart-2](flow-chart-2.png)

**Example**
```javascript
// GET — retrieve a resource
// GET /api/users/1
// Response: { id: 1, name: "Alice", email: "alice@example.com" }

// POST — create a new resource
// POST /api/users
// Body:     { name: "Bob", email: "bob@example.com" }
// Response: { id: 2, name: "Bob", email: "bob@example.com" }

// PUT — replace entire resource
// PUT /api/users/1
// Body:     { name: "Alice Smith", email: "alice.smith@example.com" }
// Response: { id: 1, name: "Alice Smith", email: "alice.smith@example.com" }

// PATCH — update specific fields only
// PATCH /api/users/1
// Body:     { email: "new@example.com" }
// Response: { id: 1, name: "Alice", email: "new@example.com" }

// DELETE — remove a resource
// DELETE /api/users/1
// Response: 204 No Content (empty body) or { message: "Deleted" }
```

---

### 3. HTTP Status Codes

**Theory**: Every response includes a status code that instantly communicates the result. Learn these ranges — they're universally used.

**Working Flow**
![flow-chart-3](flow-chart-3.png)

**Example**
```javascript
async function createUser(userData) {
  const res = await fetch("/api/users", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(userData)
  });

  if (res.status === 201) return res.json();       // created
  if (res.status === 400) throw new Error("Validation failed");
  if (res.status === 401) redirectToLogin();
  if (res.status === 409) throw new Error("Email already exists");
  if (res.status >= 500) throw new Error("Server error, try again");
}
```

---

### 4. HTTP Headers

**Theory**: Headers carry metadata about the request or response — what format the data is in, how to authenticate, what the client accepts. They're key-value pairs sent before the body.

**Working Flow**
![flow-chart-4](flow-chart-4.png)

**Example**
```javascript
// GET with auth header
const res = await fetch("/api/profile", {
  method: "GET",
  headers: {
    "Authorization": `Bearer ${localStorage.getItem("token")}`,
    "Accept": "application/json"
  }
});

// POST with content type
const res2 = await fetch("/api/posts", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "Authorization": `Bearer ${token}`
  },
  body: JSON.stringify({ title: "Hello", body: "World" })
});

// Read response headers
console.log(res2.headers.get("Content-Type")); // "application/json"
console.log(res2.headers.get("X-Request-Id")); // some server-set ID
```

---

### 5. The Fetch API — GET, POST, and Proper Error Handling

**Theory**: `fetch()` is the browser's built-in HTTP client. It returns a Promise that resolves to a Response object. Crucially, fetch only rejects on network failure — NOT on 4xx/5xx status codes. You must check `response.ok` for HTTP errors.

**Working Flow**
![flow-chart-5](flow-chart-5.png)

**Example — GET with query parameters**
```javascript
async function fetchUsers(page = 1, limit = 10, search = "") {
  const params = new URLSearchParams({ page, limit, search });
  const url = `/api/users?${params}`;
  // → /api/users?page=1&limit=10&search=alice

  const res = await fetch(url, {
    headers: {
      "Authorization": `Bearer ${getToken()}`,
      "Accept": "application/json"
    }
  });

  if (!res.ok) {
    throw new Error(`Failed to fetch users: ${res.status}`);
  }

  const data = await res.json();
  return data; // { users: [...], total: 100, page: 1 }
}
```

**Output** (console after call)
```
data.users → [{ id: 1, name: "Alice" }, { id: 2, name: "Bob" }]
data.total → 100
data.page  → 1
```

**Example — POST with body**
```javascript
async function createPost(title, body, tags) {
  const res = await fetch("/api/posts", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "Authorization": `Bearer ${getToken()}`
    },
    body: JSON.stringify({ title, body, tags })
  });

  if (res.status === 201) {
    const newPost = await res.json();
    console.log("Created:", newPost.id);
    return newPost;
  }

  if (res.status === 400) {
    const errors = await res.json();
    throw new ValidationError(errors.message);
  }

  throw new Error(`Unexpected status: ${res.status}`);
}
```

---

### 6. A Complete CRUD Example — Users API

**Theory**: Real apps need all four operations. Here's a complete, reusable API client module.

**Working Flow**
![flow-chart-6](flow-chart-6.png)

**Example**
```javascript
// api/users.js
const BASE_URL = "https://api.example.com";

function getHeaders() {
  return {
    "Content-Type": "application/json",
    "Authorization": `Bearer ${localStorage.getItem("token")}`
  };
}

async function handleResponse(res) {
  if (res.status === 204) return null; // no content
  const data = await res.json();
  if (!res.ok) throw new Error(data.message || `HTTP ${res.status}`);
  return data;
}

// Read all
export async function getUsers(params = {}) {
  const query = new URLSearchParams(params);
  const res = await fetch(`${BASE_URL}/users?${query}`, {
    headers: getHeaders()
  });
  return handleResponse(res);
}

// Read one
export async function getUser(id) {
  const res = await fetch(`${BASE_URL}/users/${id}`, {
    headers: getHeaders()
  });
  return handleResponse(res);
}

// Create
export async function createUser(userData) {
  const res = await fetch(`${BASE_URL}/users`, {
    method: "POST",
    headers: getHeaders(),
    body: JSON.stringify(userData)
  });
  return handleResponse(res); // returns created user with new ID
}

// Update (partial)
export async function updateUser(id, changes) {
  const res = await fetch(`${BASE_URL}/users/${id}`, {
    method: "PATCH",
    headers: getHeaders(),
    body: JSON.stringify(changes)
  });
  return handleResponse(res);
}

// Delete
export async function deleteUser(id) {
  const res = await fetch(`${BASE_URL}/users/${id}`, {
    method: "DELETE",
    headers: getHeaders()
  });
  return handleResponse(res); // null on 204
}
```

```javascript
// Usage
import { getUsers, createUser, updateUser, deleteUser } from './api/users.js';

try {
  const users = await getUsers({ page: 1, limit: 10 });
  console.log(users); // { data: [...], total: 50 }

  const newUser = await createUser({ name: "Alice", email: "alice@example.com" });
  console.log("Created:", newUser.id); // Created: 51

  const updated = await updateUser(51, { email: "alice.smith@example.com" });
  console.log("Updated:", updated.email); // Updated: alice.smith@example.com

  await deleteUser(51);
  console.log("Deleted");
} catch (e) {
  console.error("API error:", e.message);
}
```

**Output**
```
users.data.length  → 10
users.total        → 50
Created: 51
Updated: alice.smith@example.com
Deleted
```

---

### Real-World: REST vs GraphQL — Quick Comparison

**Working Flow**
![flow-chart-7](flow-chart-7.png)

---

[View Interview Questions](./interview.md)
