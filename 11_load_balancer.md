# 11 — Load Balancer: The Traffic Cop

## The Problem It Solves

One server can only handle so much. When too many requests hit the same machine:
- CPU spikes
- Memory runs out
- Response times shoot up
- The server crashes

A **load balancer** sits in front of multiple servers and distributes incoming requests across them.

```mermaid
flowchart TD
    U1[User 1] --> LB[Load Balancer]
    U2[User 2] --> LB
    U3[User 3] --> LB
    U4[...millions of users] --> LB
    LB --> S1[Server 1 - 33% of traffic]
    LB --> S2[Server 2 - 33% of traffic]
    LB --> S3[Server 3 - 33% of traffic]
```

Benefits:
- No single server gets overwhelmed
- Add or remove servers without clients knowing
- If a server dies, traffic automatically routes around it
- Zero-downtime deployments (take one server down, update it, bring it back)

---

## Load Balancing Algorithms

How does the load balancer decide which server gets the next request?

---

### 1. Round Robin

Requests go to each server in turn, looping around.

```mermaid
sequenceDiagram
    participant LB as Load Balancer
    participant S1 as Server 1
    participant S2 as Server 2
    participant S3 as Server 3

    LB->>S1: Request 1
    LB->>S2: Request 2
    LB->>S3: Request 3
    LB->>S1: Request 4
    LB->>S2: Request 5
    LB->>S3: Request 6
```

**Good when**: All servers are identical in power and requests are roughly equal in complexity.

**Problem**: If request 1 is a huge file upload and request 2 is a tiny health check, Server 1 is overloaded while Server 2 is idle.

---

### 2. Least Connections

Send the next request to whichever server has the **fewest active connections right now**.

```mermaid
flowchart TD
    New[New request arrives] --> Check{Which server has fewest open connections?}
    Check --> S1[Server 1: 45 connections]
    Check --> S2[Server 2: 12 connections ← pick this one]
    Check --> S3[Server 3: 38 connections]
    S2 --> Handle[Handle the request]
```

**Good when**: Requests vary significantly in how long they take to process.

---

### 3. Weighted Round Robin

Servers get different weights based on their capacity. A powerful server handles more requests.

```mermaid
flowchart LR
    LB[Load Balancer] -->|Weight 3 - 3 out of 6 requests| S1[Server 1 - 16 CPU cores]
    LB -->|Weight 2 - 2 out of 6 requests| S2[Server 2 - 8 CPU cores]
    LB -->|Weight 1 - 1 out of 6 requests| S3[Server 3 - 4 CPU cores]
```

**Good when**: Servers have different hardware specs.

---

### 4. IP Hash

The client's IP address is hashed. The result determines which server they always go to.

```mermaid
flowchart LR
    Alice[Alice - IP: 192.168.1.1] -->|hash → Server 1| S1[Server 1]
    Bob[Bob - IP: 192.168.1.2] -->|hash → Server 2| S2[Server 2]
    Alice2[Alice again - same IP] -->|same hash → Server 1| S1
```

**Good when**: You need **session stickiness** — the same user must always hit the same server (e.g., in-memory sessions without a shared cache).

---

### 5. GEO-Based Routing

Route users to the server closest to them geographically to minimize latency.

```mermaid
flowchart TD
    LB[Global Load Balancer / DNS]
    UserIndia[User in India] -->|50ms| LB
    UserUSA[User in USA] -->|50ms| LB
    UserEU[User in Europe] -->|50ms| LB
    LB --> Asia[Asia Region Servers - Mumbai]
    LB --> US[US Region Servers - Virginia]
    LB --> EU[EU Region Servers - Frankfurt]
    UserIndia -.->|routed to nearest| Asia
    UserUSA -.->|routed to nearest| US
    UserEU -.->|routed to nearest| EU
```

Used by Netflix, Google, Cloudflare, and every major global service.

---

## Summary of Algorithms

| Algorithm | How it works | Best for |
|---|---|---|
| **Round Robin** | Take turns, equally | Identical servers, similar requests |
| **Least Connections** | Pick the least busy | Long-running requests |
| **Weighted Round Robin** | Like round robin, but servers have weights | Mixed hardware capacity |
| **IP Hash** | Same client → same server | Session stickiness |
| **GEO-based** | Route to nearest server | Global apps |

---

## Health Checks

A load balancer constantly checks if servers are alive. If a server fails, the load balancer stops sending it traffic.

```mermaid
flowchart TD
    LB[Load Balancer] -->|Every 10s: GET /health| S1[Server 1]
    LB -->|Every 10s: GET /health| S2[Server 2]
    LB -->|Every 10s: GET /health| S3[Server 3]

    S1 -->|200 OK - healthy| LB
    S2 -->|200 OK - healthy| LB
    S3 -->|Connection timeout - DEAD| LB

    LB -->|Mark Server 3 as down| Out[Stop sending traffic to Server 3]
    Out --> Alert[Alert on-call engineer]
```

Types of health checks:
- **Active checks**: Load balancer pings servers on a schedule
- **Passive checks**: Load balancer watches for error responses and marks servers unhealthy after too many failures

---

## Layer 4 vs Layer 7 Load Balancing

```mermaid
graph LR
    LB[Load Balancer types] --> L4[Layer 4 - Transport Layer]
    LB --> L7[Layer 7 - Application Layer]
    
    L4 --> l4a[Routes based on IP address and TCP port only]
    L4 --> l4b[Fast - just routing, no content inspection]
    L4 --> l4c[Cannot route based on URL path or headers]
    
    L7 --> l7a[Routes based on HTTP headers, URL paths, cookies]
    L7 --> l7b[Can do SSL termination]
    L7 --> l7c[Can route /api/ to API servers and /static/ to CDN]
    L7 --> l7d[Slightly slower - more processing]
```

**Example L7 routing:**
```
/api/*    → API server pool
/static/* → CDN / static server pool
/ws/*     → WebSocket server pool
```

---

## Load Balancer in a Full Architecture

```mermaid
flowchart TD
    Internet[Internet] --> DNS[DNS - routes to nearest region]
    DNS --> GLB[Global Load Balancer - L7]
    GLB -->|/api requests| LB_API[API Load Balancer]
    GLB -->|/static requests| CDN[CDN]
    LB_API --> API1[API Server 1]
    LB_API --> API2[API Server 2]
    LB_API --> API3[API Server 3]
    API1 & API2 & API3 --> Cache[(Redis Cache)]
    Cache --> DB[(Database)]
```
