# GraphQL Interview Questions

1. **What is GraphQL?**
   - It is a query language for APIs and a runtime for fulfilling those queries with your existing data. It provides a more efficient and flexible alternative to REST.

2. **What are the main differences between REST and GraphQL?**
   - **REST**: Multiple endpoints, fixed data structure per endpoint, often leads to over-fetching or under-fetching.
   - **GraphQL**: Single endpoint, client defines exactly what data it needs, avoids over-fetching.

3. **What is "Over-fetching" and "Under-fetching"?**
   - **Over-fetching**: When an API returns more data than you actually need (e.g., getting a full user object when you only need the name).
   - **Under-fetching**: When an API doesn't return enough data, forcing you to make multiple network requests (e.g., fetching a post, then fetching the author's details in a separate call).

4. **What is a "Query" in GraphQL?**
   - A Query is used to fetch data from the server. It is the read-only operation of GraphQL.

5. **What is a "Mutation"?**
   - A Mutation is used to modify data on the server (Create, Update, or Delete).

6. **What is the "Schema" in GraphQL?**
   - The Schema is the core of any GraphQL server implementation. It describes the functionality available to clients, including the types of data, their relationships, and the operations (Queries/Mutations) that can be performed.

7. **What is a "Resolver"?**
   - A resolver is a function that retrieves the data for a specific field in a GraphQL query. It acts as the bridge between the query and the data source (database, another API, etc.).
 Riverside.
 Riverside.
