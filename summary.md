markdown
# System Design Fundamentals - Complete Guide

> Based on: Introduction to System Design (YouTube)
> 
> A step-by-step journey from single server to distributed systems architecture.

## Table of Contents

1. [Core Concepts](#core-concepts)
2. [Architecture Evolution](#architecture-evolution)
3. [The 5 Questions Framework](#the-5-questions-framework)
4. [Scaling Strategies](#scaling-strategies)
5. [Component Deep Dives](#component-deep-dives)
6. [Decision Matrix](#decision-matrix)
7. [Interview Tips](#interview-tips)

---

## Core Concepts

### What is System Design?

System design is the skill of **choosing the right components** (databases, caches, load balancers, CDNs) and **placing them correctly** to solve software problems.

**Engineering Growth Path:**
- **Junior Engineer**: Build specific features with known inputs/outputs
- **Mid-Senior Engineer**: Solve problems and fit solutions into existing systems
- **Senior Engineer**: Design the entire system and make technology choices

### HLD vs LLD

| Aspect | High-Level Design (HLD) | Low-Level Design (LLD) |
|--------|------------------------|------------------------|
| **Scope** | Big picture view | Single feature zoom |
| **Components** | App servers, databases, caches, CDNs | Class structures, function names |
| **Example** | "Design Instagram" architecture | "Design the Like button" implementation |
| **Interview Tip** | Mention components, data flow, trade-offs | Discuss data structures, algorithms |

### Functional vs Non-Functional Requirements

- **Functional**: What the app does (upload photo, follow user, like post)
- **Non-Functional**: How well it does it (speed <200ms, durability, availability, cost)

> **Key Insight**: Two apps with identical functional requirements (family photo app vs Instagram) can be completely different systems based on non-functional requirements.

---

## Architecture Evolution

### The 4 Stages of Growth

```mermaid
flowchart LR
    subgraph S1["Stage 1: Single Server (Monolith)"]
        A["Users"] --> B["Web Server\n+ App\n+ Database"]
    end

    subgraph S2["Stage 2: Load Balancer + Multiple App Servers"]
        C["Users"] --> D["Load Balancer"]
        D --> E["App Server 1"]
        D --> F["App Server 2"]
        D --> G["App Server N"]
        E --> H["Database"]
        F --> H
        G --> H
    end

    subgraph S3["Stage 3: Cache Layer + Database Replication"]
        I["Users"] --> J["Load Balancer"]
        J --> K["App Server 1"]
        J --> L["App Server 2"]
        J --> M["App Server N"]
        K --> N["Cache\n(Redis/Memcached)"]
        L --> N
        M --> N
        K --> O["Primary DB"]
        L --> O
        M --> O
        O --> P["Read Replica 1"]
        O --> Q["Read Replica 2"]
        N --> O
        N --> P
        N --> Q
    end

    subgraph S4["Stage 4: Sharding + Object Storage"]
        R["Users"] --> S["CDN"]
        S --> T["Load Balancer"]
        T --> U["App Server 1"]
        T --> V["App Server 2"]
        T --> W["App Server N"]
        U --> X["Cache Cluster"]
        V --> X
        W --> X
        U --> Y["Shard 1"]
        V --> Z["Shard 2"]
        W --> AA["Shard N"]
        U --> AB["Object Storage\n(S3/GCS)"]
        V --> AB
        W --> AB
        X --> Y
        X --> Z
        X --> AA
    end

    S1 --> S2 --> S3 --> S4
The Three Failures (And Fixes)
Failure	Symptom	Solution	Component Added
CPU/RAM Exhaustion	Pages load in 10s instead of 200ms	Distribute traffic across servers	Load Balancer
Database Overload	Same queries run repeatedly	Store popular answers in fast memory	Cache
Machine Death	Disk failure = data loss	Keep copies on multiple machines	Replication + Object Storage
The 5 Questions Framework
Before designing anything, ask:

How many users and how fast are they growing?

1,000 users vs 50 million users = completely different systems
Design for where you're heading, not where you are
Is the app read-heavy or write-heavy?

Photo app: Users scroll 1000s of photos but upload only 2-5/month
Optimize the read path (feed) over write path (upload)
What data can you absolutely never lose?

Photos: Never (durability critical)
Like counts: Tolerable to be off by 3 for a few seconds
How much latency can you afford?

Feed load: <200ms (must feel instant)
Photo upload: 2-3 seconds acceptable (feels like work)
What does it cost?

Every replica, cache, and server adds to the bill
Right design = meets requirements for least money
Scaling Strategies
Vertical vs Horizontal
mermaid
graph TB
    subgraph Vertical["Vertical Scaling"]
        V1["Small Server"] --> V2["Medium Server"] --> V3["Large Server"] --> V4["Maximum Hardware Limit"]
        style V4 fill:#ffcccc
    end
    
    subgraph Horizontal["Horizontal Scaling"]
        H1["Server 1"] 
        H2["Server 2"]
        H3["Server 3"]
        H4["Server N..."]
        LB["Load Balancer"]
        
        LB --> H1
        LB --> H2
        LB --> H3
        LB --> H4
    end
Aspect	Vertical Scaling	Horizontal Scaling
Method	Bigger server (more CPU/RAM)	More servers behind load balancer
Limit	Hardware ceiling	No upper limit
Single Point of Failure	Yes (one big server)	No (distributed)
Cost Curve	Exponential (pay more for less gain)	Linear
When to Use	Early stage, <1000 users	Growth stage, >10k users
Rule of Thumb: Scale vertically first (simple, zero code changes), then horizontally when you hit limits.

Monolith vs Microservices
Start with Monolith when:

Team size < 10 engineers
User base < 100k
Need rapid development
Want simple debugging (one log to check)
Split to Microservices when:

Need independent scaling (feed needs 10 servers, uploads need 2)
Team size > 40 engineers (avoid deployment queues)
One service needs different technology stack
Warning: A 5-person startup running 30 microservices spends more time debugging network than building product.

Component Deep Dives
1. Load Balancer
Functions:

Traffic Distribution: Round Robin, Least Connections, Weighted
Health Checks: Pings servers every few seconds; removes dead servers
Reverse Proxy: Sits in front of app servers
Algorithms:

Round Robin: Request 1 → Server 1, Request 2 → Server 2, etc.
Least Connections: Send to server with fewest active requests
Weighted: Bigger servers get more traffic
High Availability: Run multiple load balancers (if one dies, another takes over).

2. Stateless vs Stateful
The Problem:
User logs in on Server 1 (session stored in Server 1's memory). Next request goes to Server 5 → "Who are you? Please log in."

The Rule:

If this server vanished right now, would any user lose something? If yes → that data has no business living on the server.

Solution:

Move sessions to shared store (Redis)
App servers become stateless (disposable)
Can add/kill/restart servers without user impact
Anti-pattern: Sticky Sessions (pinning users to one server) - defeats the purpose of load balancing.

3. Caching Strategy
Cache Hit vs Miss:

Cache Hit: <1ms response (Redis)
Cache Miss: 20-30ms (Database query)
What to Cache:

Trending photos (read constantly, changes rarely)
Popular profile cards (Messi/Ronaldo profiles)
Login sessions (tiny, read on every request)
Cache Invalidation Strategies:

Strategy	Use Case	Trade-off
TTL (Time To Live)	Trending feed (30s), Profile (few mins)	Lazy, may show stale data
Active Invalidation	Privacy settings, security data	Immediate sync, more work
Cache Stampede:
When popular entry expires and 1000s hit database simultaneously.

Fix: Stagger TTLs (60s vs 61s vs 62s)
Fix: Allow one request to rebuild while others wait
4. Database Replication
Architecture: 1 Primary (Writes) + 2 Replicas (Reads)

mermaid
flowchart LR
    A["Write Query"] --> P["Primary DB"]
    P -->|"Replication Stream"| R1["Replica 1"]
    P -->|"Replication Stream"| R2["Replica 2"]
    
    B["Read Query"] --> LB["Read Load Balancer"]
    LB --> R1
    LB --> R2
    
    style P fill:#ffcccc,stroke:#cc0000,stroke-width:2px
    style R1 fill:#ccffcc,stroke:#00cc00,stroke-width:2px
    style R2 fill:#ccffcc,stroke:#00cc00,stroke-width:2px
Consistency Models:

Eventual Consistency: Replicas lag behind primary by milliseconds; acceptable for most reads (other users seeing your photo 500ms late)
Strong Consistency: Read from primary for user's own recent data (you seeing your upload immediately)
Replication Lag Bug:
User uploads photo → refresh profile → photo missing (read went to replica that hasn't synced yet).

Fix: "Read your own writes" from primary database
Failover:
If primary dies → promote replica to primary (automatic or manual).

⚠️ Critical Warning:
Replicas copy everything including mistakes. If someone runs DELETE on primary, replicas delete too within milliseconds. Replicas are not backups! Use actual snapshot backups stored separately.

5. Database Sharding
When: Data too big for one machine (billions of photos).

Concept: Split data across multiple databases (shards). Each shard holds a slice.

Shard Key: Field used to determine which shard (e.g., user_id % number_of_shards).

mermaid
flowchart TB
    subgraph HashRing["Consistent Hash Ring (0-360°)"]
        direction LR
        
        subgraph Shard0["Shard 0 (0-90°)"]
            S0["Shard 0"]
            U5["User ID 5"]
            U8["User ID 8"]
        end
        
        subgraph Shard1["Shard 1 (90-180°)"]
            S1["Shard 1"]
            U13["User ID 13"]
            U15["User ID 15"]
        end
        
        subgraph Shard2["Shard 2 (180-270°)"]
            S2["Shard 2"]
            U10["User ID 10"]
            U22["User ID 22"]
        end
        
        subgraph Shard3["Shard 3 (270-360°)"]
            S3["Shard 3"]
            U30["User ID 30"]
            U35["User ID 35"]
        end
    end
    
    subgraph HotShardWarning["⚠️ Hot Shard Alert"]
        warning["Celebrity users concentrated<br/>on one shard causing imbalance"]
    end
    
    hash["hash(User ID) % 360"]
    
    hash -->|"hash(10)=190°"| U10
    hash -->|"hash(13)=120°"| U13
    hash -.->|"hash(100)=135°"| warning
    
    style Shard0 fill:#e1f5fe
    style Shard1 fill:#fff3e0
    style Shard2 fill:#e8f5e9
    style Shard3 fill:#f3e5f5
    style HotShardWarning fill:#ffebee,stroke:#c62828,stroke-width:2px
    style warning fill:#ffcdd2,stroke:#c62828
    style S1 fill:#ffcc80,stroke:#e65100,stroke-width:3px
Problems:

Hot Shard: Celebrities (Messi, Ronaldo) land on same shard → overload

Fix: Better shard key that distributes evenly (consistent hashing)
Resharding Nightmare: Adding 5th shard changes user_id % 4 to % 5 → almost all data moves

Fix: Consistent Hashing - only small range of data moves when adding shards
Decision Matrix
Decision	Option A	Option B	When to Choose
Architecture	Monolith	Microservices	Start Monolith; split when >40 engineers or need independent scaling
Scaling	Vertical	Horizontal	Vertical first; Horizontal when hitting hardware limits
Database	SQL (PostgreSQL)	NoSQL (MongoDB)	SQL for structured relational data; NoSQL for unstructured/varying schema
Session Storage	Sticky Sessions	Shared Store (Redis)	Always choose Shared Store for true horizontal scaling
Cache Invalidation	TTL	Active Invalidation	TTL for tolerable staleness; Active for security/privacy
Consistency	Eventual	Strong	Eventual for others' data; Strong for own recent writes
Interview Tips
The Golden Opening
When asked "Design Instagram," start with:

"I'll stay at the high level. I'll mention components, data flow, and trade-offs, and we can zoom in later if you want."

This shows you understand HLD vs LLD distinction.

Common Interview Scenarios
Scenario 1: "Users getting logged out after scaling to 5 servers"

Cause: Login sessions stored in local memory (stateful)
Fix: Move sessions to shared store (Redis)
Bonus: Mention sticky sessions and why they're wrong (uneven load, server death = session loss)
Scenario 2: "Cache is full, what do you evict?"

Answer: LRU (Least Recently Used) - evict items not accessed longest
Scenario 3: "Hot entry expires, 10,000 requests hit DB"

Term: Cache Stampede
Fix: Stagger TTLs, allow one request to rebuild while others wait
Scenario 4: "SQL vs NoSQL for photo app?"

Answer: SQL because data is relational (users, photos, follows connected). Need joins for feed generation.
Bonus: Mention using both - SQL for core data, NoSQL for user preferences/analytics
Critical Concepts to Mention
ACID: Atomicity, Consistency, Isolation, Durability (SQL guarantee)
Eventual Consistency: Replicas catch up eventually (acceptable for most social media)
Single Point of Failure: Every component that can take down the system (mitigate with redundancy)
Replication Lag: Time delay between primary and replicas (causes "where's my photo?" bugs)
The "Read Your Own Writes" Pattern
Always route user's own recent data reads to the primary database:

User uploads photo → write to primary
User refreshes to see photo → read from primary (not replica)
Other users viewing that photo → can read from replica (eventual consistency OK)
Complete Request Flow
mermaid
flowchart TD
    A["User Device"] --> B["DNS Lookup<br/>(photoapp.com → IP)"]
    B --> C["Load Balancer"]
    C --> D["App Server<br/>(Stateless)"]
    D --> E{"Cache Check<br/>(Redis)"}
    
    E -->|"Cache Hit"| J["Return Cached Data<br/>(<1ms)"]
    E -->|"Cache Miss"| F{"Read or Write?"}
    
    F -->|"Read"| G["Read from<br/>PostgreSQL Replicas"]
    F -->|"Write"| H["Write to<br/>PostgreSQL Primary"]
    
    G --> I["Object Storage<br/>(S3)"]
    H --> I
    
    I --> K["Update Redis Cache"]
    
    J --> L["Format Response"]
    K --> L
    
    L --> M["App Server"]
    M --> N["Load Balancer"]
    N --> O["User Device"]
    
    style A fill:#e3f2fd
    style O fill:#e8f5e9
    style H fill:#ffebee,stroke:#c62828
    style G fill:#e8f5e9,stroke:#2e7d32
Key Takeaways
Start Simple: Single server is correct for most apps initially
Add Components When Needed: Don't preemptively optimize; add load balancers, caches, replicas when metrics show need
Every Fix Creates New Problems: Cache adds invalidation complexity; replication adds lag; sharding makes joins harder
Stateless Servers: Keep servers disposable; move state to shared stores
Ask Questions First: The first 5 minutes of a design interview should be clarifying requirements, not drawing boxes
Remember: System design is not about memorizing diagrams. It's about understanding trade-offs and making logical choices based on requirements.

