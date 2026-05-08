# GraphQL API — Interview Questions

---

**1. What is GraphQL and how is it different from REST?**

GraphQL is a query language for APIs with a single endpoint where the client describes exactly what data it needs. REST uses multiple fixed endpoints that return server-defined data shapes.

```text
REST:
  GET /users/1          → full user object (over-fetch)
  GET /users/1/posts    → second request needed (under-fetch)

GraphQL:
  POST /graphql  →  { user(id:"1") { name posts { title } } }
  Response       →  exactly { name, posts:[{title}] }  — one request
```

Key differences:
- **Over-fetching**: REST returns all fields; GraphQL returns only requested fields
- **Under-fetching**: REST needs multiple requests; GraphQL resolves nested data in one
- **Versioning**: REST needs `/v1`, `/v2`; GraphQL is additive — just add new fields
- **Type safety**: GraphQL schema defines types; REST has no enforced contract

---

**2. What is the GraphQL Schema and why does it matter?**

The schema is the contract between client and server — a typed definition of all data types, their fields, and available operations.

```graphql
type User {
  id: ID!          # ! = required (non-null)
  name: String!
  email: String!
  posts: [Post!]!
}

type Query {
  user(id: ID!): User   # returns null if not found
  posts: [Post!]!       # never null, never contains nulls
}

type Mutation {
  createUser(name: String!, email: String!): User!
}
```

The schema serves as live documentation, enables editor autocomplete, and allows the GraphQL engine to validate queries before sending them to resolvers.

---

**3. What is the difference between a Query, Mutation, and Subscription?**

```graphql
# Query — read data (like GET)
query GetUser($id: ID!) {
  user(id: $id) { name email }
}

# Mutation — write data (like POST/PUT/DELETE)
mutation CreatePost($title: String!) {
  createPost(title: $title) { id title }
}

# Subscription — real-time push (like WebSocket)
subscription OnPostAdded {
  postAdded { id title author { name } }
}
```

- **Query**: Fetches data, safe to call multiple times (idempotent read)
- **Mutation**: Creates, updates, or deletes data — causes side effects
- **Subscription**: Opens a persistent connection; server pushes events to client

---

**4. What are over-fetching and under-fetching?**

**Over-fetching**: API returns more data than the client needs. Wastes bandwidth.
```text
GET /users/1  →  { id, name, email, phone, address, company, website, ... }
You only needed: name — the rest is wasted data transferred over the network.
```

**Under-fetching**: API returns too little, forcing extra requests (the N+1 problem).
```text
GET /posts          →  [{ id, title, userId }, ...]   (no author name)
GET /users/1        →  second request per post!
GET /users/2        →  third request...
= N+1 requests for N posts
```

GraphQL eliminates both: you ask for `{ posts { title author { name } } }` and get exactly that in one request.

---

**5. What is a Resolver in GraphQL?**

A resolver is the function that provides the data for a specific field in the schema. Every field in a GraphQL response is backed by a resolver.

```javascript
const resolvers = {
  Query: {
    // resolver for Query.user
    user: (parent, args, context) => {
      return context.db.users.findById(args.id); // fetch from DB
    },
    posts: (parent, args, context) => context.db.posts.findAll(),
  },
  User: {
    // resolver for User.posts — called for each user returned
    posts: (user, args, context) => {
      return context.db.posts.findByAuthor(user.id);
    },
  },
  Mutation: {
    createPost: (parent, args, context) => {
      return context.db.posts.create(args);
    },
  },
};
```

---

**6. How does Apollo Client's caching work?**

Apollo Client uses a normalized in-memory cache. Each object is stored by its `__typename + id` key. Repeated queries for the same object return cached data instantly.

```jsx
// First call → network request
const { data } = useQuery(GET_USER, { variables: { id: "1" } });
// Returns from network, stores in cache: { "User:1": { name, email, ... } }

// Second call (different component) → cache hit, no network
const { data } = useQuery(GET_USER, { variables: { id: "1" } });
// Returns immediately from cache

// After mutation, invalidate cache:
const [createPost] = useMutation(CREATE_POST, {
  refetchQueries: [{ query: GET_USER, variables: { id: "1" } }],
  // OR use cache.modify() for fine-grained optimistic updates
});
```

Cache policies:
- `cache-first` (default): serve cache, skip network if data exists
- `network-only`: always fetch fresh data
- `cache-and-network`: serve cache immediately, then update from network

---

**7. What are GraphQL Fragments and when do you use them?**

Fragments are reusable field selections. Use them when multiple queries need the same set of fields — avoids duplication.

```graphql
fragment PostPreview on Post {
  id
  title
  publishedAt
  author { name }
}

# Reuse in multiple queries
query GetFeed {
  posts { ...PostPreview }
}

query GetUserPosts($userId: ID!) {
  user(id: $userId) {
    posts { ...PostPreview }
  }
}
```

In Apollo Client with React, fragments also enable component co-location — each component declares the fields it needs.

---

**8. How do you handle errors in GraphQL?**

GraphQL can return both `data` and `errors` in the same response (partial success). Always check both.

```javascript
const { data, errors } = await graphqlFetch(QUERY, variables);

// Apollo Client:
const { data, error } = useQuery(GET_USER, { variables: { id } });

if (error) {
  // error.networkError   → network failure (no response)
  // error.graphQLErrors  → server returned errors array
  error.graphQLErrors.forEach(e => console.error(e.message, e.path));
}
```

```text
Partial error response:
{
  "data":   { "user": { "name": "Richa", "posts": null } },
  "errors": [{ "message": "Not authorized to view posts", "path": ["user","posts"] }]
}
→ You can still display the user's name while showing "posts unavailable"
```

---

**9. How do you perform authentication with Apollo Client?**

Add an auth header to every request using an `authLink` middleware:

```javascript
import { ApolloClient, InMemoryCache, createHttpLink } from "@apollo/client";
import { setContext } from "@apollo/client/link/context";

const httpLink = createHttpLink({ uri: "/graphql" });

const authLink = setContext((_, { headers }) => {
  const token = localStorage.getItem("token");
  return {
    headers: {
      ...headers,
      authorization: token ? `Bearer ${token}` : "",
    },
  };
});

const client = new ApolloClient({
  link: authLink.concat(httpLink),   // chain: auth → http
  cache: new InMemoryCache(),
});
```

Every request now automatically includes the JWT token from localStorage.

---

**10. What is the N+1 problem in GraphQL and how is it solved?**

The N+1 problem: if you fetch 10 posts and each post's resolver fetches its author separately, you make 1 + 10 = 11 database queries.

```text
Query: { posts { title author { name } } }

Naive resolvers:
  SELECT * FROM posts              → 1 query, returns 10 posts
  SELECT * FROM users WHERE id=1  → 1 query per post
  SELECT * FROM users WHERE id=2
  ... (10 more queries!)
  Total: 11 queries
```

Solution: **DataLoader** — batches and deduplicates database calls.

```javascript
import DataLoader from "dataloader";

const userLoader = new DataLoader(async (userIds) => {
  // Called ONCE with all IDs collected across this request
  const users = await db.users.findAll({ where: { id: userIds } });
  return userIds.map(id => users.find(u => u.id === id));
});

// Resolver
const resolvers = {
  Post: {
    author: (post) => userLoader.load(post.userId), // batched!
  },
};
// Result: 1 posts query + 1 users query (with IN clause) = 2 total
```
