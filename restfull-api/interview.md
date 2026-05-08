# RESTful API Interview Questions

1. **What does REST stand for?**
   - Representational State Transfer.

2. **What are the main HTTP methods used in REST?**
   - **GET**: Retrieve data.
   - **POST**: Create new data.
   - **PUT**: Update/Replace data.
   - **PATCH**: Partially update data.
   - **DELETE**: Remove data.

3. **What is the difference between PUT and PATCH?**
   - **PUT** replaces the entire resource with the new data.
   - **PATCH** only updates specific fields of the resource.

4. **What does "Stateless" mean in REST?**
   - It means that the server does not store any information about the client's previous requests. Every request must contain all the information necessary for the server to understand and process it (e.g., an auth token).

5. **Explain common status code ranges.**
   - **2xx**: Success (e.g., 200 OK, 201 Created).
   - **3xx**: Redirection.
   - **4xx**: Client Error (e.g., 404 Not Found, 401 Unauthorized).
   - **5xx**: Server Error (e.g., 500 Internal Server Error).

6. **What are "Idempotent" methods?**
   - An idempotent method is one that can be called multiple times without changing the result beyond the initial application. **GET**, **PUT**, and **DELETE** are idempotent. **POST** is NOT idempotent (calling it twice creates two resources).

7. **How do you handle authentication in REST?**
   - Typically through the **Authorization** header using tokens like **JWT** (JSON Web Tokens) or API keys.
 Riverside.
 Riverside.
