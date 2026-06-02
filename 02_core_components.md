# 02 — Core Components of System Design

## The Lego Bricks of Every Big System

No matter what you're building — Instagram, Netflix, WhatsApp, a banking app — they're all made from the same basic building blocks. The art of system design is knowing which blocks to use and how to connect them.

```mermaid
graph TD
    subgraph "What the user sees"
        Client[Client App - Browser / Mobile]
    end

    subgraph "The middle"
        LB[Load Balancer]
        Server1[Backend Server 1]
        Server2[Backend Server 2]
        Cache[Cache - Redis/Memcached]
        MQ[Message Queue]
    end

    subgraph "Storage"
        DB[(Primary Database)]
        Replica[(DB Replica)]
        FileStore[(File/Object Storage)]
    end

    subgraph "Visibility"
        Monitor[Monitoring & Logs]
    end

    Client --> LB
    LB --> Server1
    LB --> Server2
    Server1 --> Cache
    Server2 --> Cache
    Cache --> DB
    DB --> Replica
    Server1 --> MQ
    MQ --> Worker[Background Worker]
    Worker --> DB
    Server1 --> Monitor
    Server2 --> Monitor
```

---

## 1. Client Applications

The part the user actually touches.

| Type | Examples | Characteristics |
|---|---|---|
| Web browser | Chrome, Firefox | Downloads HTML/CSS/JS, runs in browser |
| Mobile app | iOS, Android | Native code, installed on device |
| Desktop app | Electron apps, native apps | Installed locally |
| IoT device | Smart thermostat, sensor | Minimal compute, sends data |

The client **sends requests** and **receives responses**. It does very little heavy lifting — all the real work happens on the server.

```mermaid
sequenceDiagram
    participant User
    participant Client as Client App
    participant Server as Backend Server

    User->>Client: Taps "Post Photo"
    Client->>Server: POST /upload (photo data)
    Server-->>Client: 201 Created - photo URL
    Client-->>User: Shows photo in feed
```

---

## 2. Backend Servers

The brain of the operation. This is where your **business logic** lives.

What a backend server does:
- Receives requests from clients
- Validates the input
- Runs the logic (e.g., "does this user have enough balance?")
- Reads from or writes to the database
- Returns a response

```mermaid
flowchart LR
    Request[Incoming Request] --> Validate[Validate input]
    Validate --> Auth[Check authentication]
    Auth --> Logic[Run business logic]
    Logic --> DB[(Read / Write DB)]
    DB --> Format[Format response]
    Format --> Response[Send response]
```

A single server has limits: CPU, memory, and network bandwidth. When too many requests arrive at once, the server slows down and eventually crashes. That's why we need multiple servers and a load balancer in front.

---

## 3. Databases

Where all the data lives — permanently (unlike RAM which disappears when the server restarts).

Two broad types:

```mermaid
graph LR
    DB[Database] --> SQL[SQL - Relational]
    DB --> NoSQL[NoSQL - Non-relational]

    SQL --> s1[Tables with rows and columns]
    SQL --> s2[Strong consistency]
    SQL --> s3[Good for structured data]
    SQL --> s4[MySQL, PostgreSQL, SQLite]

    NoSQL --> n1[Flexible schema]
    NoSQL --> n2[Scales horizontally easier]
    NoSQL --> n3[Key-value, document, graph]
    NoSQL --> n4[MongoDB, Redis, Cassandra]
```

Databases are often the **bottleneck** in systems. Everything waits on the database. That's why so much of system design is about reducing database load (caching), spreading the data (sharding), and keeping backups (replication).

---

## 4. Load Balancers

Imagine a restaurant with 3 chefs but 1 waiter who only ever hands orders to chef 1. Chef 1 is overwhelmed. Chef 2 and 3 are bored. That's what happens without a load balancer.

A **load balancer** sits in front of your servers and distributes incoming requests evenly.

```mermaid
flowchart TD
    Users[1000 users/second] --> LB[Load Balancer]
    LB -->|~333 req/s| S1[Server 1]
    LB -->|~333 req/s| S2[Server 2]
    LB -->|~333 req/s| S3[Server 3]
```

Benefits:
- No single server gets crushed
- If one server dies, traffic is routed to healthy ones
- You can add more servers without the client knowing

---

## 5. Message Queues

Sometimes you don't need to do things immediately. If a user uploads a photo, you don't need to resize it, generate thumbnails, and send email notifications all in the same second they hit upload. You can do those things **later**, in the background.

A **message queue** is like a to-do list for your backend workers.

```mermaid
flowchart LR
    Server[API Server] -->|adds job| Queue[(Message Queue)]
    Queue -->|picks up job| W1[Worker 1 - resize image]
    Queue -->|picks up job| W2[Worker 2 - send email]
    Queue -->|picks up job| W3[Worker 3 - update feed]
```

Benefits:
- The API responds instantly to the user
- Heavy work happens asynchronously
- If a worker crashes, the job stays in the queue and gets retried
- Workers can be scaled independently

---

## 6. Monitoring and Logs

You can't fix what you can't see. Monitoring is the system's pulse check — always running in the background, alerting you when something is wrong.

```mermaid
flowchart TD
    S1[Server 1] -->|emits metrics| Monitor[Monitoring System]
    S2[Server 2] -->|emits metrics| Monitor
    DB[(Database)] -->|query times| Monitor
    LB[Load Balancer] -->|request counts| Monitor

    Monitor --> Dashboard[Dashboard - Grafana / Datadog]
    Monitor -->|threshold breached| Alert[Alert - PagerDuty / Slack]
    Monitor --> Logs[Log Storage - Elasticsearch]
```

What you monitor:
- **Latency**: how long requests take
- **Throughput**: how many requests per second
- **Error rate**: % of requests that fail
- **CPU / Memory / Disk**: is the machine healthy?
- **Database query times**: is something slow?

---

## How They All Fit Together

Here's a simplified view of a photo-sharing app (like Instagram):

```mermaid
flowchart TD
    App[Mobile App] -->|HTTPS| CDN[CDN - serve static assets]
    App -->|API calls| LB[Load Balancer]
    LB --> API1[API Server 1]
    LB --> API2[API Server 2]
    API1 --> Cache[(Redis Cache)]
    API2 --> Cache
    Cache -->|cache miss| DB[(PostgreSQL)]
    DB --> Replica[(Read Replica)]
    API1 --> Queue[Message Queue - Kafka]
    Queue --> Worker[Image Processing Worker]
    Worker --> S3[(Object Storage - photos)]
    API1 --> Monitor[Monitoring]
    API2 --> Monitor
```

Every component in this diagram has one job. Each one can be scaled, upgraded, or replaced independently.
