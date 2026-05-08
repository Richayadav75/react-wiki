- Category: Backend / API
- Track: Web Development
- Difficulty: Intermediate
- Related: restful-api, api-calls-react

### What is GraphQL?
**GraphQL** is a query language for your API and a server-side runtime for executing those queries by using a type system you define for your data. Unlike REST, which has multiple endpoints, GraphQL usually has a **single endpoint**.

---

### 1. Data Selection Flow
**Working Flow: Requesting EXACTLY what you need**

```mermaid
graph LR
    A[Client App] -->|Query: { name, age }| B[GraphQL Endpoint]
    B -->|Response: { name, age }| A
    C[Another Client] -->|Query: { name, email }| B
    B -->|Response: { name, email }| C
```

---

### 2. GraphQL vs REST

| Feature | REST | GraphQL |
| :--- | :--- | :--- |
| **Endpoints** | Multiple (`/users`, `/posts`) | **Single** (`/graphql`) |
| **Data Fetching** | Over-fetching or Under-fetching | **Precise** (Get exactly what you ask) |
| **Structure** | Fixed by Server | Defined by Client Query |
| **Versioning** | Requires new URLs (v1, v2) | **Versionless** (Just add new fields) |

---

### 3. Core Concepts

#### The Schema
A definition of the data types and relationships available in the API.

#### Queries (Read)
Used to fetch data. You describe the fields you want.
```graphql
query {
  user(id: "1") {
    name
    email
  }
}
```

#### Mutations (Write)
Used to create, update, or delete data.
```graphql
mutation {
  addUser(name: "Richa") {
    id
  }
}
```

---

### 4. Comprehensive Example (Apollo Client)
**Theory**: While you can use `fetch`, libraries like **Apollo Client** make it much easier to integrate GraphQL with React.
```tsx
const GET_USER = gql`
  query GetUser($id: ID!) {
    user(id: $id) { name }
  }
`;

function User({ id }) {
  const { loading, error, data } = useQuery(GET_USER, { variables: { id } });

  if (loading) return 'Loading...';
  return <h1>{data.user.name}</h1>;
}
```

---

### 5. Summary: Why use GraphQL?
1. **Efficiency**: Reduce bandwidth by avoiding over-fetching.
2. **Speed**: Get multiple resources in a single round-trip.
3. **Developer Experience**: Strong typing and excellent tooling (GraphiQL).

---

[View Interview Questions](./interview.md)
