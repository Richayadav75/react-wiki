- Category: API / Data Fetching
- Difficulty: Intermediate
- Related: api-calls-react, restful-api, async-api

### GraphQL APIs
GraphQL is a query language for your API. Instead of the server deciding what data to return, the **client specifies exactly what fields it wants**. One endpoint handles all operations — reads, writes, and real-time subscriptions.

**Analogy**
REST is a fixed-price buffet — you get the whole plate whether you're hungry or not. GraphQL is a la carte dining — you order exactly what you want and nothing more. One waiter handles all orders (single endpoint), and the kitchen (resolver) fetches just your dishes.

---

### 1. REST vs GraphQL — The Over/Under-Fetching Problem
**Theory**: In REST, each endpoint returns a fixed shape of data. If you need a user's name and their post titles, REST requires 2 requests and returns far more fields than you need. GraphQL solves both problems in one request.

**Working Flow**
![flow-chart](flow-chart.png)

**Example — Same Data, Two Approaches**
```javascript
// ----- REST -----
const userRes  = await fetch("/api/users/1");
const user     = await userRes.json();          // full user object (20+ fields)

const postsRes = await fetch("/api/users/1/posts");
const posts    = await postsRes.json();         // full post objects

const display  = { name: user.name, titles: posts.map(p => p.title) }; // finally

// ----- GraphQL -----
const query = `
  query {
    user(id: "1") {
      name
      posts { title }
    }
  }
`;
const res  = await fetch("/graphql", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ query }),
});
const { data } = await res.json();
// data = { user: { name: "Leanne", posts: [{ title: "..." }] } }
```

**Output**
```
REST:
  Request 1 → { id:1, name:"Leanne", email:"...", address:{...}, ... } (20 fields)
  Request 2 → [{id:1,title:"...",body:"...",userId:1}, ...] (100 posts, all fields)

GraphQL:
  Request 1 → { user: { name:"Leanne", posts:[{title:"..."},{title:"..."},...] } }
```

| Feature | REST | GraphQL |
| :--- | :--- | :--- |
| Endpoints | Many (`/users`, `/posts`, `/comments`) | Single (`/graphql`) |
| Data shape | Fixed by server | Defined by client |
| Over-fetching | Common | Impossible |
| Under-fetching | Common (N+1 problem) | Solved in one query |
| Versioning | `/v1/`, `/v2/` | Additive (just add fields) |

---

### 2. GraphQL Schema, Types, and Resolvers
**Theory**: The Schema is the contract between client and server. It defines all available types, their fields, and what queries and mutations can be run. Resolvers are the functions that fulfil each field.

**Working Flow**
![flow-chart-2](flow-chart-2.png)

**Example — Schema Definition (SDL)**
```graphql
# Types
type User {
  id: ID!              # ! means required (non-null)
  name: String!
  email: String!
  posts: [Post!]!      # array of Posts, never null
}

type Post {
  id: ID!
  title: String!
  body: String!
  author: User!
}

# Root Query type — what clients can read
type Query {
  user(id: ID!): User
  posts: [Post!]!
}

# Root Mutation type — what clients can write
type Mutation {
  createPost(title: String!, body: String!, authorId: ID!): Post!
  deletePost(id: ID!): Boolean!
}

# Subscription — real-time events
type Subscription {
  postAdded: Post!
}
```

**Output**
```
Schema defines the "menu" of operations.
Resolver for user(id) → looks up DB row → returns User object
Resolver for User.posts → fetches all posts WHERE authorId = user.id
Client gets exactly: { user: { name, posts: [{ title }] } }
```

---

### 3. Queries — Reading Data with Variables
**Theory**: A GraphQL Query is the read operation. Use variables (prefixed with `$`) instead of hardcoding values in the query string — this keeps queries reusable and prevents injection attacks.

**Working Flow**
![flow-chart-3](flow-chart-3.png)

**Example**
```graphql
# Query definition (write once, reuse with different variables)
query GetUserProfile($id: ID!) {
  user(id: $id) {
    name
    email
    posts {
      id
      title
    }
  }
}
```

```javascript
// Send query + variables together
const response = await fetch("/graphql", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    query: GET_USER_PROFILE_QUERY,
    variables: { id: "42" },    // injected safely
  }),
});

const { data, errors } = await response.json();
// data.user = { name: "Richa", email: "r@example.com", posts: [...] }
// errors    = undefined (or array of error objects)
```

**Output**
```
variables: { id: "42" }
→ data: {
    user: {
      name: "Richa Yadav",
      email: "richa@example.com",
      posts: [
        { id: "1", title: "Getting started with GraphQL" },
        { id: "2", title: "React + Apollo tutorial" }
      ]
    }
  }
```

---

### 4. Mutations — Writing Data
**Theory**: Mutations are GraphQL's write operations (Create, Update, Delete). They look like queries but start with the `mutation` keyword. They can also return data, so you can update your UI with the new record in one round trip.

**Working Flow**
![flow-chart-4](flow-chart-4.png)

**Example**
```graphql
mutation CreatePost($title: String!, $body: String!, $authorId: ID!) {
  createPost(title: $title, body: $body, authorId: $authorId) {
    id
    title
    author { name }
  }
}
```

```javascript
const { data } = await fetch("/graphql", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    query: CREATE_POST_MUTATION,
    variables: {
      title: "My First Post",
      body: "Hello GraphQL world!",
      authorId: "42",
    },
  }),
}).then(r => r.json());

// data.createPost = { id: "99", title: "My First Post", author: { name: "Richa" } }
```

**Output**
```
mutation variables → { title, body, authorId }
server creates row in DB
→ returns: { id: "99", title: "My First Post", author: { name: "Richa Yadav" } }
UI immediately shows new post (no second GET request)
```

---

### 5. Apollo Client in React — `useQuery` and `useMutation`
**Theory**: Apollo Client is the most popular GraphQL client for React. It handles request sending, caching, loading/error states, and re-fetching. `useQuery` replaces `fetch + useEffect`. `useMutation` replaces `fetch + setState` for writes.

**Working Flow**
![flow-chart-5](flow-chart-5.png)

**Example**
```jsx
import { ApolloClient, InMemoryCache, ApolloProvider, gql, useQuery, useMutation } from "@apollo/client";

// 1. Setup client (done once in index.js / main.jsx)
const client = new ApolloClient({
  uri: "https://api.example.com/graphql",
  cache: new InMemoryCache(),
});

// Wrap your app
// <ApolloProvider client={client}><App /></ApolloProvider>

// 2. Define queries with gql tag (enables syntax highlighting + parsing)
const GET_USER = gql`
  query GetUser($id: ID!) {
    user(id: $id) {
      name
      email
      posts { id title }
    }
  }
`;

const CREATE_POST = gql`
  mutation CreatePost($title: String!, $authorId: ID!) {
    createPost(title: $title, authorId: $authorId) {
      id
      title
    }
  }
`;

// 3. Use in components
function UserProfile({ userId }) {
  const { loading, error, data } = useQuery(GET_USER, {
    variables: { id: userId },
  });

  if (loading) return <p>Loading profile...</p>;
  if (error)   return <p>Error: {error.message}</p>;

  const { user } = data;
  return (
    <div>
      <h2>{user.name} ({user.email})</h2>
      <h3>Posts</h3>
      <ul>{user.posts.map(p => <li key={p.id}>{p.title}</li>)}</ul>
    </div>
  );
}

function NewPostForm({ authorId }) {
  const [title, setTitle] = React.useState("");
  const [createPost, { loading }] = useMutation(CREATE_POST, {
    refetchQueries: [{ query: GET_USER, variables: { id: authorId } }],
  });

  return (
    <form onSubmit={e => {
      e.preventDefault();
      createPost({ variables: { title, authorId } });
      setTitle("");
    }}>
      <input value={title} onChange={e => setTitle(e.target.value)} />
      <button type="submit" disabled={loading}>
        {loading ? "Posting..." : "Add Post"}
      </button>
    </form>
  );
}
```

**Output**
```
Loading profile...
→ Richa Yadav (richa@example.com)
  Posts:
  • Getting started with GraphQL
  • React + Apollo tutorial

[input field] [Add Post]
→ "Posting..."
→ list auto-refreshes with new post (refetchQueries)
```

---

### 6. Fragments and Error Handling
**Theory**: Fragments let you define a reusable set of fields that you can spread into multiple queries. GraphQL errors come back in an `errors` array alongside `data` — both can exist at the same time (partial success).

**Working Flow**
![flow-chart-6](flow-chart-6.png)

**Example**
```graphql
# Fragment — define once, use everywhere
fragment UserFields on User {
  id
  name
  email
}

query GetUser($id: ID!) {
  user(id: $id) {
    ...UserFields          # spread the fragment
    posts { title }
  }
}

mutation UpdateUser($id: ID!, $name: String!) {
  updateUser(id: $id, name: $name) {
    ...UserFields          # reuse same fragment in mutation response
  }
}
```

```javascript
// Error handling — GraphQL can return BOTH data and errors
const { data, errors } = await graphqlFetch(query, variables);

if (errors) {
  errors.forEach(err => {
    console.error(`[GraphQL error] ${err.message}`);
    // err.locations = [{ line: 3, column: 5 }]
    // err.path = ["user", "posts", 0]
  });
}
if (data?.user) {
  // partial data may still be usable
}
```

**Output**
```
Partial error response example:
{
  "data": { "user": { "name": "Richa", "posts": null } },
  "errors": [{ "message": "Cannot fetch posts: permission denied", "path": ["user","posts"] }]
}
→ Show user name ✓, show "posts unavailable" message for posts section
```

---

### Real-World Example: User Profile with Nested Posts (Single Query)
```jsx
const GET_PROFILE = gql`
  query GetProfile($userId: ID!) {
    user(id: $userId) {
      name
      email
      avatarUrl
      posts(limit: 5) {
        id
        title
        publishedAt
        commentsCount
      }
    }
  }
`;

function ProfilePage({ userId }) {
  const { loading, error, data, refetch } = useQuery(GET_PROFILE, {
    variables: { userId },
    notifyOnNetworkStatusChange: true,
  });

  if (loading) return <div className="skeleton">Loading profile...</div>;
  if (error) return (
    <div>
      <p>Failed to load: {error.message}</p>
      <button onClick={() => refetch()}>Try Again</button>
    </div>
  );

  const { user } = data;

  return (
    <div className="profile">
      <img src={user.avatarUrl} alt={user.name} />
      <h1>{user.name}</h1>
      <p>{user.email}</p>

      <section>
        <h2>Recent Posts</h2>
        {user.posts.map(post => (
          <article key={post.id}>
            <h3>{post.title}</h3>
            <small>{post.publishedAt} · {post.commentsCount} comments</small>
          </article>
        ))}
      </section>
    </div>
  );
}
```

**Output**
```
(loading)  → [skeleton placeholder]

(success)  → [avatar image]
             Richa Yadav
             richa@example.com

             Recent Posts
             ┌──────────────────────────────────┐
             │ Getting started with GraphQL      │
             │ Jan 15, 2025 · 12 comments        │
             ├──────────────────────────────────┤
             │ React + Apollo tutorial           │
             │ Feb 2, 2025 · 5 comments          │
             └──────────────────────────────────┘
             
Note: achieved in ONE network request — no N+1 problem
```

---

[View Interview Questions](./interview.md)
