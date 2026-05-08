# Async (Real-time) API Interview Questions

1. **What is the main difference between REST and WebSockets?**
   - **REST**: Stateless, one-way communication (request-response). The connection is closed after each response.
   - **WebSockets**: Stateful, bidirectional (two-way) communication. The connection remains open, allowing both parties to send data at any time.

2. **What are Server-Sent Events (SSE)?**
   - SSE is a technology that allows a server to push real-time updates to a web page over HTTP. Unlike WebSockets, it is strictly one-way (server-to-client).

3. **When would you choose SSE over WebSockets?**
   - Choose SSE for scenarios that only require a one-way data stream from the server (like a live news feed or a stock ticker). SSE is easier to implement, works over standard HTTP, and has automatic reconnection built-in.

4. **What is "Long Polling"?**
   - It is a technique where the client requests data from the server, and the server holds the request open until it has new information to send. Once the client receives the data, it immediately sends another request.

5. **What is Socket.io?**
   - It is a JavaScript library that enables real-time, bidirectional communication. It uses WebSockets when available but provides fallbacks to other techniques (like long polling) if the browser or network doesn't support them.

6. **What are the challenges of scaling real-time APIs?**
   - Since connections are persistent, servers must manage thousands of open connections simultaneously, which consumes more memory and CPU than traditional stateless APIs. You often need tools like Redis to sync messages across multiple server instances.

7. **How do you handle authentication in WebSockets?**
   - Since WebSockets don't have a standard header for auth after the initial handshake, you typically pass a token during the connection request or send an "auth" message immediately after the connection is established.
 Riverside.
 Riverside.
