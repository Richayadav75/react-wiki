# RESTful API Interview Questions

---

**1. What does REST stand for and what are its core principles?**

REST = Representational State Transfer. It is an architectural style (not a protocol) for networked applications. Core constraints:
- **Stateless** — each request contains all info needed; server stores no client session
- **Uniform Interface** — resources identified by URLs, accessed via standard HTTP verbs
- **Client-Server** — frontend and backend are independent; they communicate only via the API
- **Cacheable** — responses declare whether they can be cached
- **Layered System** — client doesn't know if it talks to real server, CDN, or load balancer

---

**2. What is the difference between PUT and PATCH?**

- `PUT` replaces the entire resource. Any fields not included in the body are deleted/reset to defaults.
- `PATCH` updates only the specified fields. Other fields remain unchanged.

```
// Resource: { id: 1, name: "Alice", email: "a@x.com", role: "user" }

PUT /users/1  body: { name: "Alice Smith", email: "a@x.com" }
// Result: { id: 1, name: "Alice Smith", email: "a@x.com", role: undefined }
// role was not included → lost

PATCH /users/1  body: { email: "new@x.com" }
// Result: { id: 1, name: "Alice", email: "new@x.com", role: "user" }
// only email changed, rest preserved
```

---

**3. Why does fetch not reject on 404 or 500?**

`fetch()` only rejects the promise if the request itself failed (no network, DNS error). A server responding with 404 or 500 is still a successful HTTP exchange — the promise resolves with a Response object. You must check `response.ok` (true for 200-299) or `response.status` to detect HTTP errors:

```javascript
const res = await fetch("/api/user/999");
// res.ok is false, res.status is 404 — but no throw!

if (!res.ok) {
  throw new Error(`HTTP ${res.status}`); // you must throw manually
}
const data = await res.json();
```

---

**4. What is the difference between 401 and 403 status codes?**

- `401 Unauthorized` — you are not authenticated. You need to provide credentials (log in, include a token). The server doesn't know who you are.
- `403 Forbidden` — you are authenticated (server knows who you are) but you don't have permission to access this resource.

```
GET /admin/dashboard (no token)      → 401 Unauthorized
GET /admin/dashboard (user token)    → 403 Forbidden (need admin role)
GET /admin/dashboard (admin token)   → 200 OK
```

---

**5. What does "idempotent" mean and which HTTP methods are idempotent?**

An operation is idempotent if calling it multiple times produces the same result as calling it once. It doesn't accumulate effects on repeated calls.

| Method | Idempotent? | Reason |
|---|---|---|
| GET | Yes | Reading data never changes it |
| PUT | Yes | Replacing with same data = same result |
| DELETE | Yes | Deleting again still results in "it's gone" |
| PATCH | No | Depends on implementation (e.g., increment fields) |
| POST | No | Calling twice creates two resources |

Important for retry logic: only safe to auto-retry idempotent requests after a network failure.

---

**6. How do you send authentication credentials in REST API calls?**

Use the `Authorization` request header. Two common patterns:

**Bearer token (JWT):**
```javascript
fetch("/api/profile", {
  headers: {
    "Authorization": `Bearer ${localStorage.getItem("token")}`
  }
});
```

**Basic auth (username:password base64 encoded — avoid in production over HTTP):**
```javascript
const credentials = btoa("username:password");
fetch("/api/data", {
  headers: {
    "Authorization": `Basic ${credentials}`
  }
});
```

For session-based auth, cookies are sent automatically — no manual header needed.

---

**7. What is the Content-Type header and when do you need it?**

`Content-Type` tells the server what format the request body is in. Required whenever you send a body (POST, PUT, PATCH):

```javascript
// JSON body
fetch("/api/users", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ name: "Alice" })
});

// Form data (file upload)
const formData = new FormData();
formData.append("avatar", fileInput.files[0]);
fetch("/api/upload", {
  method: "POST",
  // NO Content-Type header for FormData — browser sets it with boundary automatically
  body: formData
});
```

---

**8. What are query parameters vs path parameters and when to use each?**

- **Path parameters** (`/users/42`) — identify a specific resource. Used when the value is required to locate the resource.
- **Query parameters** (`/users?role=admin&page=2`) — filter, sort, or paginate a collection. Optional context for a request.

```
GET /users/42           → get user with ID 42 (path param)
GET /users?role=admin   → filter users by role (query param)
GET /posts/5/comments   → comments for post 5 (nested resource)
GET /posts?author=5&sort=date&limit=10  → filters + pagination
```

---

**9. What is the difference between REST and GraphQL?**

| | REST | GraphQL |
|---|---|---|
| Endpoints | Multiple (one per resource) | Single `/graphql` |
| Data shape | Fixed by server | Client specifies exact fields needed |
| Over-fetching | Common (gets all fields) | Never (ask only what you need) |
| Under-fetching | Common (need multiple requests) | One request for nested data |
| File upload | Easy (multipart/form-data) | Needs workarounds |
| Caching | Easy (HTTP caching by URL) | More complex (query-based) |
| Learning curve | Low | Higher |

Use REST for simple CRUD APIs. Use GraphQL for complex, nested data with many clients that need different views of the same data.

---

**10. How would you implement CRUD operations for a "posts" resource?**

Design the URL structure first:

```
GET    /api/posts              → list posts (with filters)
GET    /api/posts/:id          → get one post
POST   /api/posts              → create post (returns 201)
PUT    /api/posts/:id          → replace post
PATCH  /api/posts/:id          → update post fields
DELETE /api/posts/:id          → delete post (returns 204)
```

Implementation:

```javascript
const BASE = "/api/posts";
const headers = () => ({
  "Content-Type": "application/json",
  "Authorization": `Bearer ${getToken()}`
});

const api = {
  list: (params) => fetch(`${BASE}?${new URLSearchParams(params)}`, { headers: headers() }).then(r => r.json()),
  get:  (id) => fetch(`${BASE}/${id}`, { headers: headers() }).then(r => r.json()),
  create: (data) => fetch(BASE, { method: "POST", headers: headers(), body: JSON.stringify(data) }).then(r => r.json()),
  update: (id, data) => fetch(`${BASE}/${id}`, { method: "PATCH", headers: headers(), body: JSON.stringify(data) }).then(r => r.json()),
  delete: (id) => fetch(`${BASE}/${id}`, { method: "DELETE", headers: headers() }).then(r => r.status === 204 ? null : r.json())
};

// Usage
const posts = await api.list({ page: 1, limit: 10, category: "tech" });
const post = await api.create({ title: "Hello", body: "World" });
await api.update(post.id, { title: "Updated Title" });
await api.delete(post.id);
```
