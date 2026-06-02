# 06 — APIs and Communication: How Systems Talk to Each Other

## What is an API?

An **API (Application Programming Interface)** is a contract. It says:

> "If you send me *this*, I'll give you *that*."

It's how two completely separate systems communicate without knowing each other's internal details.

```mermaid
flowchart LR
    App[Your App] -->|"Give me user #42"| API[API]
    API -->|"Here's user #42 data"| App
    
    note1[You don't know HOW it gets the data]
    note2[You just know the contract]
```

Real-world analogy: A TV remote. You press "Volume Up" and the TV gets louder. You don't need to know what's inside the TV. The remote's buttons are the API.

---

## The 5 Major API Styles

```mermaid
graph TD
    API[API Styles] --> REST[REST - most common]
    API --> SOAP[SOAP - old enterprise]
    API --> GraphQL[GraphQL - flexible queries]
    API --> gRPC[gRPC - fast microservices]
    API --> WS[WebSockets - real-time]
```

---

## 1. REST (Representational State Transfer)

The most widely used style. Uses standard HTTP methods and URLs to represent resources.

**Mental model**: A file system in the cloud. Each URL is a "path" to a resource.

```mermaid
flowchart LR
    GET[GET /users/42] -->|Read user 42| Response[{"id": 42, "name": "Alice"}]
    POST[POST /users] -->|Create new user| Response2[{"id": 43, "name": "Bob"}]
    PUT[PUT /users/42] -->|Replace user 42| Response3[Updated user]
    DELETE[DELETE /users/42] -->|Delete user 42| Response4[204 No Content]
```

| Feature | REST |
|---|---|
| Protocol | HTTP/HTTPS |
| Data format | JSON (usually) |
| Learning curve | Low |
| Cacheability | Excellent (GET requests are cacheable) |
| Best for | Public APIs, web and mobile apps |

---

## 2. SOAP (Simple Object Access Protocol)

The old-school way. Still used in banking and enterprise software. Uses XML envelopes.

```mermaid
flowchart LR
    Client -->|XML envelope via HTTP or SMTP| SOAP[SOAP Server]
    SOAP -->|XML response| Client
```

**Example SOAP request** (just to show how verbose it is):
```xml
<Envelope xmlns="http://schemas.xmlsoap.org/soap/envelope/">
  <Body>
    <GetUser>
      <UserId>42</UserId>
    </GetUser>
  </Body>
</Envelope>
```

| Feature | SOAP |
|---|---|
| Protocol | HTTP, SMTP, or others |
| Data format | XML (strict, verbose) |
| Learning curve | High |
| Best for | Enterprise, banking, systems requiring strict contracts |
| Still used? | Yes — in legacy systems and financial services |

---

## 3. GraphQL

Invented by Facebook (2015). Lets the client ask for **exactly the data it needs** — no more, no less.

**Problem with REST**: If you want a user's name and their last 3 posts, you might need 2 separate API calls:
- `GET /users/42` → returns 20 fields, you only wanted 2
- `GET /users/42/posts?limit=3`

**GraphQL solves this**:

```graphql
query {
  user(id: 42) {
    name
    posts(limit: 3) {
      title
      createdAt
    }
  }
}
```

One request. Exactly the data you asked for.

```mermaid
flowchart LR
    Client -->|One query, ask for what you need| GQL[GraphQL Server /graphql]
    GQL -->|fetches from multiple sources| DB[(Database)]
    GQL -->|fetches from multiple sources| MicroS[Other Microservices]
    GQL -->|Returns exactly what was asked| Client
```

| Feature | GraphQL |
|---|---|
| Protocol | HTTP |
| Data format | JSON |
| Best for | Mobile apps, complex frontends, when different clients need different data |
| Downside | Caching is harder, more complex to set up |

---

## 4. gRPC (Google Remote Procedure Call)

Think of it as calling a function on another machine as if it were local.

Instead of "send me JSON and I'll parse it", gRPC says "call this exact method with these exact parameters".

```mermaid
flowchart LR
    ServiceA[Service A] -->|"GetUser(userId=42)"| ServiceB[Service B]
    ServiceB -->|Returns User struct| ServiceA
```

Uses **Protocol Buffers** (protobuf) — a binary format. Much smaller and faster than JSON.

| Feature | gRPC |
|---|---|
| Protocol | HTTP/2 |
| Data format | Binary (Protocol Buffers) |
| Speed | Very fast — 5-10x faster than REST |
| Best for | Internal microservice communication |
| Downside | Binary format is not human-readable, harder to debug |

---

## 5. WebSockets

REST/gRPC are **request-response**: client asks, server answers, connection closes.

WebSockets keep the connection **permanently open** so the server can push data to the client at any time.

```mermaid
sequenceDiagram
    participant Client
    participant Server

    Client->>Server: HTTP Upgrade Request (switch to WebSocket)
    Server-->>Client: 101 Switching Protocols

    Note over Client,Server: Persistent two-way connection open

    Server-->>Client: "New message from Alice"
    Server-->>Client: "Bob is typing..."
    Client->>Server: "Send message: Hello"
    Server-->>Client: "Message delivered"
```

| Feature | WebSockets |
|---|---|
| Connection type | Persistent, bidirectional |
| Best for | Chat apps, live feeds, multiplayer games, collaborative editing |
| Examples | WhatsApp Web, Google Docs, stock price tickers |
| Downside | Harder to scale (connections stay open), not cacheable |

---

## Comparison Table

| | REST | SOAP | GraphQL | gRPC | WebSockets |
|---|---|---|---|---|---|
| Direction | Request-Response | Request-Response | Request-Response | Request-Response | Bidirectional |
| Protocol | HTTP | HTTP/SMTP/etc | HTTP | HTTP/2 | WS |
| Format | JSON | XML | JSON | Binary | Any |
| Speed | Medium | Slow | Medium | Fast | Real-time |
| Caching | Easy | Hard | Hard | Hard | N/A |
| Best for | Public APIs | Enterprise legacy | Flexible clients | Microservices | Real-time apps |

---

## When to Use What

```mermaid
flowchart TD
    Q{What are you building?} --> A[Public-facing web API]
    Q --> B[Real-time feature like chat or live updates]
    Q --> C[Internal service-to-service communication]
    Q --> D[Mobile app with complex data needs]
    Q --> E[Banking or enterprise legacy integration]

    A --> REST[Use REST]
    B --> WS[Use WebSockets]
    C --> GRPC[Use gRPC]
    D --> GQL[Use GraphQL]
    E --> SOAP[Use SOAP]
```
