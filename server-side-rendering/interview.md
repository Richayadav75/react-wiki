# Server-Side Rendering Interview Questions

1. **What is Server-Side Rendering (SSR)?**
   - HTML is generated on the server for each request and sent fully formed to the browser. The browser displays content immediately without waiting for JavaScript to build the page. React then "hydrates" the HTML to make it interactive.

2. **What is the difference between SSR, CSR, SSG, and ISR?**
   - **CSR** (Client-Side Rendering) — browser downloads blank HTML + JS, JS builds the page. **SSR** — server generates HTML per request. **SSG** (Static Site Generation) — HTML built at deploy time, served as static files. **ISR** (Incremental Static Regeneration) — SSG pages that can be automatically regenerated in the background at an interval.

3. **What are the advantages of SSR over CSR?**
   - Faster time-to-first-visible-content (browser displays pre-built HTML), better SEO (crawlers receive full HTML content), better performance on slow devices (less JS to parse and execute on client).

4. **What is hydration in React?**
   - After the browser displays the SSR HTML, React runs on the client and attaches event listeners to the existing DOM nodes without re-creating them. This makes the static-looking page interactive.

5. **What is a hydration mismatch and how do you fix it?**
   - When the HTML the server renders differs from what React would render on the client — causing a flicker or error. Common causes: using `Date`, `Math.random()`, or `window`/`localStorage` directly in render. Fix: defer client-only code to `useEffect`.
   ```jsx
   // ❌ Different on server vs client
   <p>{new Date().toLocaleTimeString()}</p>

   // ✅ Only runs on client after hydration
   const [time, setTime] = useState('');
   useEffect(() => { setTime(new Date().toLocaleTimeString()); }, []);
   ```

6. **What is getServerSideProps in Next.js?**
   - A function exported from a page file that runs on the server for every request. It has access to the request object (cookies, headers) and returns data as props to the page component. Used for auth-protected or always-fresh pages.

7. **What is getStaticProps in Next.js?**
   - Runs at build time, fetches data, and returns it as props. The page is pre-rendered as static HTML. Great for pages where data doesn't change often (blog posts, marketing pages).

8. **What is ISR and how does the revalidate option work?**
   - ISR allows static pages to be regenerated in the background after a set interval. `revalidate: 60` means the page can be regenerated at most once every 60 seconds. The first user after the interval triggers a rebuild; all users get the cached version until the rebuild completes.

9. **What is the SEO advantage of SSR over CSR?**
   - Search engine crawlers receive fully rendered HTML with all content visible. CSR sends a blank page with a script tag — the crawler may not wait for JavaScript to execute, meaning your content is invisible to search engines.

10. **When would you choose SSG over SSR?**
    - SSG when: content is the same for all users, changes rarely (blog, docs, marketing). SSR when: content is user-specific (dashboard, profile), requires fresh data on every request (inventory, prices), or needs access to request cookies for authentication.
