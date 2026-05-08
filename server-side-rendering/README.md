- Category: React / Performance
- Difficulty: Intermediate
- Related: virtual-dom, api-calls-react, protected-routes

### Server-Side Rendering — Generate HTML on the Server
Server-Side Rendering (SSR) means HTML is generated on the **server** for each request and sent fully formed to the browser. The browser displays content immediately (no waiting for JavaScript to build the page), then React "hydrates" the HTML to make it interactive.

**Analogy**
A restaurant with two service models. CSR (Client-Side Rendering) = you get a blank plate and a recipe — you have to cook the food yourself at the table before you can eat. SSR = you get a fully plated dish — you eat immediately. The waiter (server) does the cooking for each customer on demand.

---

### 1. CSR vs SSR — The Core Difference

**Theory**
| | CSR | SSR |
| :--- | :--- | :--- |
| HTML generation | Browser (client) | Server |
| Initial load | Blank HTML → JS runs → content appears | Full HTML → browser displays instantly |
| Time to first byte | Fast (small HTML) | Slower (server does work) |
| Time to visible content | Slow (JS must load + run) | Fast (HTML already there) |
| SEO | Poor (crawlers see blank page) | Excellent (full HTML for crawlers) |
| Server cost | Low | Higher |
| Use case | Dashboards, admin tools | Public pages, blogs, e-commerce |

**Working Flow**

![flow-chart](flow-chart.png)

**Example**
```
CSR flow:
  Browser requests / → server sends blank HTML + <script>bundle.js</script>
  Browser downloads bundle.js (200-500kb)
  React runs, fetches data from API
  React renders HTML into DOM
  → User sees content after 2-4 seconds

SSR flow:
  Browser requests / → server runs React, fetches data, renders full HTML
  Server sends complete HTML → browser displays it instantly
  React hydrates (attaches event listeners)
  → User sees content after 0.3-0.8 seconds
```

---

### 2. SSG — Static Site Generation

**Theory**
SSG generates HTML at **build time** (not per request). The HTML is pre-built and served as static files from a CDN. This is the fastest possible delivery — no server computation per request.

Perfect for: documentation, blogs, marketing pages, portfolios.

**Working Flow**

![flow-chart-2](flow-chart-2.png)

**Example**
```jsx
// Next.js — pages/blog/[slug].jsx

// getStaticPaths — tells Next.js which pages to pre-build
export async function getStaticPaths() {
  const res   = await fetch('https://api.example.com/posts');
  const posts = await res.json();

  return {
    paths: posts.map(post => ({
      params: { slug: post.slug },
    })),
    fallback: false, // 404 for paths not in list
  };
}

// getStaticProps — fetches data at BUILD TIME for each path
export async function getStaticProps({ params }) {
  const res  = await fetch(`https://api.example.com/posts/${params.slug}`);
  const post = await res.json();

  return {
    props: { post },
  };
}

// The page component receives data as props
export default function BlogPost({ post }) {
  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  );
}
```

**Output**
```
// At build time (npm run build):
├── blog/
│   ├── hello-world/index.html       ← pre-built
│   ├── react-hooks-guide/index.html ← pre-built
│   └── css-tips/index.html          ← pre-built

// User requests /blog/hello-world:
→ CDN serves static hello-world/index.html instantly (no server computation)
→ Time to first byte: ~50ms
```

---

### 3. SSR in Next.js — getServerSideProps

**Theory**
`getServerSideProps` runs on the server for **every request**. It has access to the request object (cookies, headers, query params) making it ideal for personalized or authenticated pages.

**Working Flow**

![flow-chart-3](flow-chart-3.png)

**Example**
```jsx
// Next.js — pages/dashboard.jsx

// Runs on the server for every request to /dashboard
export async function getServerSideProps(context) {
  const { req, res, params, query } = context;

  // Access cookies (for auth)
  const token = req.cookies['auth-token'];

  if (!token) {
    return {
      redirect: {
        destination: '/login',
        permanent: false,
      },
    };
  }

  // Fetch user-specific data
  const userData = await fetch('https://api.example.com/me', {
    headers: { Authorization: `Bearer ${token}` },
  }).then(r => r.json());

  return {
    props: {
      user: userData,
    },
  };
}

export default function Dashboard({ user }) {
  return (
    <div>
      <h1>Welcome back, {user.name}!</h1>
      <p>Last login: {user.lastLogin}</p>
    </div>
  );
}
```

**Output**
```
// Request to /dashboard with valid auth cookie:
Server fetches user data → renders HTML → sends to browser

// Request to /dashboard with NO cookie:
Redirects to /login (302)

// Browser receives full HTML:
<h1>Welcome back, Richa!</h1>
<p>Last login: 2024-05-15</p>
```

---

### 4. ISR — Incremental Static Regeneration

**Theory**
ISR is the best of SSG and SSR. Pages are pre-built like SSG, but can be **automatically regenerated** in the background after a set interval (`revalidate` in seconds). The first user after the interval triggers a rebuild; subsequent users get the fresh version.

**Working Flow**

![flow-chart-4](flow-chart-4.png)

**Example**
```jsx
// Next.js — pages/products/[id].jsx

export async function getStaticPaths() {
  // Pre-build the top 100 products
  const products = await fetch('/api/products?limit=100').then(r => r.json());

  return {
    paths: products.map(p => ({ params: { id: String(p.id) } })),
    fallback: 'blocking', // other products built on first request
  };
}

export async function getStaticProps({ params }) {
  const product = await fetch(`/api/products/${params.id}`).then(r => r.json());

  return {
    props: { product },
    revalidate: 60, // ← regenerate this page at most every 60 seconds
  };
}

export default function ProductPage({ product }) {
  return (
    <div>
      <h1>{product.name}</h1>
      <p>Price: ₹{product.price}</p>
      <p>Stock: {product.stock} units</p>
    </div>
  );
}
```

**Output**
```
// Build time:
Pre-built 100 product pages

// User visits /products/1 at 10:00am:
Served from cache (static, instant)

// Price updates on server at 10:00:30am
// User visits /products/1 at 10:01:05am (after 60s revalidate):
Served stale page + background regeneration triggered

// Next user gets fresh page with new price:
Price: ₹750  ← updated
```

---

### 5. Hydration — Making SSR HTML Interactive

**Theory**
After the browser receives the SSR HTML and displays it, React runs on the client and "hydrates" — it attaches event listeners to the existing HTML without re-creating DOM nodes. This is what makes SSR pages interactive.

**Hydration mismatch** happens when the server-rendered HTML doesn't match what React would render on the client — causing a flicker or error.

**Working Flow**

![flow-chart-5](flow-chart-5.png)

**Example**
```jsx
// ❌ Hydration mismatch — using Date in SSR
function Timestamp() {
  return <p>Current time: {new Date().toLocaleTimeString()}</p>;
}
// Server renders: "Current time: 10:00:00 AM"
// Client hydrates at: "Current time: 10:00:01 AM"  ← MISMATCH → error!

// ✅ Fix — only render time-sensitive data on client
import { useState, useEffect } from 'react';

function Timestamp() {
  const [time, setTime] = useState('');

  useEffect(() => {
    setTime(new Date().toLocaleTimeString());
  }, []);

  return <p>Current time: {time || '...'}</p>;
}

// ❌ Another common mismatch — window/localStorage in SSR
function ThemeButton() {
  const theme = localStorage.getItem('theme'); // ← crashes on server!
  return <button>{theme}</button>;
}

// ✅ Fix — guard with typeof window check or useEffect
function ThemeButton() {
  const [theme, setTheme] = useState('light');

  useEffect(() => {
    setTheme(localStorage.getItem('theme') || 'light');
  }, []);

  return <button>{theme}</button>;
}
```

---

### 6. CSR vs SSR vs SSG vs ISR — When to Use What

**Working Flow**

![flow-chart-6](flow-chart-6.png)

**Example**
```
Real-world e-commerce app strategy:

Page                    Strategy    Reason
─────────────────────────────────────────────────────────────────────
/                       SSG         Home page, same for everyone, CDN-fast
/products               SSG + ISR   Product catalog, revalidate every 5 min
/products/[id]          SSG + ISR   Per-product, revalidate every 60s
/cart                   CSR         Fully personalized, no SEO needed
/checkout               SSR         Needs auth token + fresh inventory check
/dashboard              SSR         User-specific, protected, always fresh
/blog/[slug]            SSG         Articles don't change, perfect for static
/search?q=...           SSR         Query-dependent, needs fresh index

Rule of thumb:
  Same for everyone + rarely changes  → SSG
  Same for everyone + updates often   → SSG + ISR
  User-specific or auth-required      → SSR
  Highly interactive, no SEO          → CSR
```

---

[View Interview Questions](./interview.md)
