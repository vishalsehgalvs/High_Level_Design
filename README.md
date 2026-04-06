# ðŸ—ï¸ System Design â€” From Zero to Planet Scale

> *"System design is just common sense at scale. The hard part is that common sense stops being obvious when millions of things happen at once."*

---

```
â•”â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•—
â•‘                                                                              â•‘
â•‘   You       â†’   1 server   â†’   works great                                  â•‘
â•‘   100 users â†’   1 server   â†’   still fine                                   â•‘
â•‘   10K users â†’   1 server   â†’   getting slow                                 â•‘
â•‘   1M users  â†’   1 server   â†’   ðŸ’¥ server is on fire                         â•‘
â•‘   1B users  â†’   ???        â†’   this is what system design solves             â•‘
â•‘                                                                              â•‘
â•šâ•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•
```

When you build a small app for yourself, you don't think about any of this â€” one laptop, one database, done.
But the moment real people start using your thing, everything breaks in ways you never imagined.
**System design is how you think through those problems before they happen.**

---

## Table of Contents

1. [What even is System Design?](#1-what-even-is-system-design)
2. [The Building Blocks â€” the Lego bricks of every big system](#2-the-building-blocks--the-lego-bricks-of-every-big-system)
3. [Scalability â€” how do you grow without breaking?](#3-scalability--how-do-you-grow-without-breaking)
4. [Availability â€” how do you stay up when things go wrong?](#4-availability--how-do-you-stay-up-when-things-go-wrong)
5. [CAP Theorem â€” the impossible triangle](#5-cap-theorem--the-impossible-triangle)
6. [Databases â€” choosing the right storage](#6-databases--choosing-the-right-storage)
7. [Caching â€” stop asking the same question twice](#7-caching--stop-asking-the-same-question-twice)
8. [Load Balancers â€” the traffic cops](#8-load-balancers--the-traffic-cops)
9. [CDN â€” the world is big, servers are slow](#9-cdn--the-world-is-big-servers-are-slow)
10. [Message Queues â€” don't do things right now](#10-message-queues--dont-do-things-right-now)
11. [API Gateway â€” the front door](#11-api-gateway--the-front-door)
12. [Rate Limiting â€” slowing down the greedy](#12-rate-limiting--slowing-down-the-greedy)
13. [Database Sharding â€” cutting the data pie](#13-database-sharding--cutting-the-data-pie)
14. [Replication â€” keeping backups alive](#14-replication--keeping-backups-alive)
15. [Microservices vs Monolith â€” one big thing or many small things?](#15-microservices-vs-monolith--one-big-thing-or-many-small-things)
16. [Real World Examples â€” how the big apps actually work](#16-real-world-examples--how-the-big-apps-actually-work)
17. [The System Design Interview Playbook](#17-the-system-design-interview-playbook)
18. [Key Numbers Every Designer Should Know](#18-key-numbers-every-designer-should-know)
19. [Glossary](#19-glossary)
20. [Single Server Setup â€” where every system starts](#20-single-server-setup--where-every-system-starts)
21. [API Design â€” the contract between systems](#21-api-design--the-contract-between-systems)
22. [API Protocols â€” the languages systems use to talk](#22-api-protocols--the-languages-systems-use-to-talk)

---

## 1. What even is System Design?

### The pizza shop analogy

Imagine you open a pizza shop.

Day 1 â€” it's just you. You take the order, make the pizza, and deliver it. Simple.

Now imagine your shop goes viral on Instagram. Suddenly 500 people want pizza at the same time. What happens?

- You can't take 500 orders yourself
- Your one oven can't cook 500 pizzas simultaneously
- You only have one delivery guy
- If your oven breaks, the whole shop shuts down

**System design is figuring out â€” before this happens â€” how you'll handle it.**

You'd hire more staff, buy more ovens, hire more delivery drivers, and set up a proper order management system. You'd think about: what if the cash register breaks? What if delivery guy 3 calls in sick? The answers are your "system design".

Now replace "pizza shop" with "Instagram" or "Google" and you get the idea.

---

### The journey from 1 user to 1 billion users

What actually breaks at each scale â€” and how you fix it:

```
â•”â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•—
â•‘  USERS         WHAT'S HAPPENING            WHAT BREAKS / WHAT YOU ADD   â•‘
â• â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•£
â•‘  1 - 100    â†’  Laptop as a server          Nothing breaks. Enjoy it.    â•‘
â•‘                                                                          â•‘
â•‘  1K         â†’  Cloud server (single)       Getting a bit slow on        â•‘
â•‘                                            read-heavy queries            â•‘
â•‘                                            FIX: Add caching (Redis)     â•‘
â•‘                                                                          â•‘
â•‘  10K        â†’  Traffic spikes unpredictably Server sometimes crashes    â•‘
â•‘                                            FIX: Load balancer +         â•‘
â•‘                                            2-3 servers                  â•‘
â•‘                                                                          â•‘
â•‘  100K       â†’  DB is the bottleneck        Queries take seconds         â•‘
â•‘                                            FIX: Read replicas,          â•‘
â•‘                                            better indexing              â•‘
â•‘                                                                          â•‘
â•‘  1M         â†’  Everything is slow          Static files slow, DB huge   â•‘
â•‘                                            FIX: CDN for static,         â•‘
â•‘                                            DB sharding                  â•‘
â•‘                                                                          â•‘
â•‘  10M        â†’  One codebase is unmaintainable Teams blocking each other â•‘
â•‘                                            FIX: Microservices,          â•‘
â•‘                                            message queues               â•‘
â•‘                                                                          â•‘
â•‘  100M+      â†’  Global users, data laws     Latency, compliance          â•‘
â•‘                                            FIX: Multi-region deploy,    â•‘
â•‘                                            geo-sharding                 â•‘
â•šâ•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•
```

---

### What a request actually does when you type a URL

Before anything else, let's trace what happens when you type `google.com` and press Enter. This flow is the foundation of everything in system design.

```mermaid
sequenceDiagram
    participant Y as ðŸ§‘ You (Browser)
    participant D as ðŸŒ DNS Server
    participant C as ðŸ“¦ CDN
    participant LB as âš–ï¸ Load Balancer
    participant S as ðŸ–¥ï¸ App Server
    participant CA as âš¡ Cache (Redis)
    participant DB as ðŸ—„ï¸ Database

    Y->>D: "What's the IP address of google.com?"
    D-->>Y: "It's 142.250.80.46"
    Y->>C: "Give me google.com's homepage"
    C-->>Y: "Here's the cached HTML/CSS/images" (fast!)

    Note over Y,DB: For dynamic data (search results, your Gmail):
    Y->>LB: "Search for 'system design'"
    LB->>S: Forwards to least-busy server
    S->>CA: "Do I have cached results for this?"
    CA-->>S: "Nope, not cached"
    S->>DB: "SELECT * FROM search_index WHERE..."
    DB-->>S: Results
    S->>CA: "Cache these results for 5 minutes"
    S-->>LB: Here are the results
    LB-->>Y: Here are your search results
```

**Every step in this diagram is a concept in system design.** The rest of this document explains each one in depth.

---

## 2. The Building Blocks â€” the Lego bricks of every big system

Every large system â€” Twitter, Uber, Netflix â€” is assembled from the same set of building blocks. Master these and you can design almost anything.

```mermaid
graph TD
    User["ðŸ‘¤ User\n(Browser / Mobile App)"]
    DNS["ðŸŒ DNS\nThe phone book of the internet\nConverts names to addresses"]
    CDN["ðŸ“¦ CDN\nCity-level cache\nServes static files near users"]
    LB["âš–ï¸ Load Balancer\nTraffic cop\nSplits work across servers"]
    API["ðŸšª API Gateway\nFront door\nAuth + routing + rate limiting"]
    S1["ðŸ–¥ï¸ App Server 1"]
    S2["ðŸ–¥ï¸ App Server 2"]
    S3["ðŸ–¥ï¸ App Server 3"]
    Cache["âš¡ Cache  Redis \nPost-it notes\nFast memory for hot data"]
    DB["ðŸ—„ï¸ Primary DB\nThe truth\nAll writes go here"]
    DBR["ðŸ—„ï¸ Replica DBs\nBackup readers\nAccepts read queries only"]
    Queue["ðŸ“¬ Message Queue\nTo-do list\nJobs processed at own pace"]
    Worker["âš™ï¸ Worker\nBackground processor\nDoes the heavy lifting async"]
    Storage["ðŸª£ Object Storage\nWarehouse\nImages, videos, large files"]

    User -->|"types URL"| DNS
    User <-->|"images, CSS, JS, videos"| CDN
    DNS -->|"returns IP"| LB
    LB -->|"routes request"| API
    API --> S1
    API --> S2
    API --> S3
    S1 <-->|"read/write"| Cache
    S2 <-->|"read/write"| Cache
    S3 <-->|"read/write"| Cache
    Cache -->|"cache miss"| DB
    DB -->|"replicates data"| DBR
    S1 -->|"queue a job"| Queue
    Queue -->|"delivers job"| Worker
    Worker -->|"writes result"| DB
    S1 <-->|"upload/download files"| Storage
```

---

### What each block does â€” with real analogies

| Block | What it does | Real-world analogy |
|---|---|---|
| **DNS** | Converts `facebook.com` into `157.240.22.35` so computers can find each other | Phone book â€” you look up "Alice" to get her number |
| **CDN** | Keeps copies of images, videos, CSS in servers physically close to users | A newspaper printing facility in every city â€” you don't ship all papers from one place |
| **Load Balancer** | Sits in front of your servers and divides incoming traffic across them | Airport check-in desk â€” "Counter 3 is free, please go there" |
| **API Gateway** | Single entry point for all traffic â€” checks auth, routes to the right service | Hotel reception â€” one desk handles room keys, complaints, and directions |
| **App Server** | Runs your actual business logic â€” processes requests, makes decisions | A chef in the kitchen who processes an order |
| **Cache** | Fast in-memory store (RAM) for frequently accessed data | Post-it notes on your desk â€” faster than going to the filing cabinet |
| **Primary DB** | The permanent home of all your data. Handles reads AND writes | The actual filing cabinet â€” source of truth |
| **Replica DB** | A copy of the primary DB that handles read queries only | A photocopy of important documents for reference |
| **Message Queue** | Holds tasks that need to be done, workers pick them up at their own pace | A restaurant ticket printer â€” orders pile up, chefs process them one by one |
| **Worker** | A background process that eats from the queue | The line cook who handles tickets |
| **Object Storage** | Cheap, scalable storage for large files (images, videos, backups) | A warehouse â€” not fast, but holds everything |

---

## 3. Scalability â€” how do you grow without breaking?

**Scalability** = your system's ability to handle more load by adding resources â€” without rewriting everything.

There are exactly two approaches:

---

### Vertical Scaling â€” make the one machine bigger

You have one server. It's getting slow. So you buy a bigger, more powerful server.

```
         BEFORE                          AFTER

    â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”            â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
    â”‚    Server       â”‚    â”€â”€â”€â”€â”€â”€â–º â”‚    BIGGER Server            â”‚
    â”‚  â– â–  2 CPU cores â”‚            â”‚  â– â– â– â– â– â– â– â– â– â– â– â– â– â–  32 cores    â”‚
    â”‚  â–‘â–‘â–‘ 8GB RAM    â”‚            â”‚  â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘â–‘ 256GB RAM â”‚
    â”‚  ðŸ’¾ 500GB SSD   â”‚            â”‚  ðŸ’¾ðŸ’¾ðŸ’¾ðŸ’¾ 8TB SSD           â”‚
    â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜            â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
       handles 1K req/s               handles 20K req/s
```

**When is this the right call?**
- You're just getting started and want simplicity
- Your problem is a single slow database query that needs more RAM
- You need a quick fix before a product launch

**When does it stop working?**
- There's a physical ceiling â€” you can't buy a server with unlimited RAM
- It's expensive disproportionately â€” doubling RAM doesn't just cost double
- It's a **single point of failure** â€” if this one giant machine dies, everything is down

---

### Horizontal Scaling â€” add more machines

Instead of making one machine bigger, you add more identical machines and split the load between them.

```
         BEFORE                          AFTER

    â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”       â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
    â”‚    Server       â”‚  â”€â”€â”€â–º â”‚ Server 1 â”‚  â”‚ Server 2 â”‚  â”‚ Server 3 â”‚
    â”‚    (struggling) â”‚       â”‚ 8GB RAM  â”‚  â”‚ 8GB RAM  â”‚  â”‚ 8GB RAM  â”‚
    â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜       â””â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”˜  â””â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”˜  â””â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”˜
                                   â”‚              â”‚             â”‚
                              â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
                                              â”‚
                                    â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
                                    â”‚   Load Balancer     â”‚
                                    â”‚  (splits traffic)   â”‚
                                    â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
                                              â”‚
                                         All traffic
```

**Why this is better at scale:**
- No ceiling â€” just keep adding machines
- One server dying doesn't kill the app â€” others pick up the slack
- Cheap commodity hardware instead of expensive specialised servers
- You can scale different parts independently

**The one catch: your app must be stateless**

```
âŒ Stateful server (bad for horizontal scaling):
   User logs in â†’ Session stored on Server 1
   Next request â†’ Load balancer sends to Server 2
   Server 2: "Who are you? I've never seen you before"
   User is logged out. Bad experience.

âœ… Stateless server (works with horizontal scaling):
   User logs in â†’ Session stored in shared Redis cache
   Next request â†’ Goes to ANY server
   Any server: "Let me check Redis... yes, you're logged in"
   Works perfectly.
```

> **The key insight:** Stateless servers don't remember you between requests. All "memory" lives in a shared external store (Redis, a database). This is what makes horizontal scaling possible.

---

### When to use which

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  START HERE                                                       â”‚
â”‚                                                                   â”‚
â”‚  Is your app in early stage / small team?                        â”‚
â”‚          â”‚                                                        â”‚
â”‚     YES â”€â”¤â”€â”€â–º Vertical scaling. Simple, no overhead.            â”‚
â”‚          â”‚                                                        â”‚
â”‚     NO   â”‚                                                        â”‚
â”‚          â”‚                                                        â”‚
â”‚  Is the bottleneck one specific thing (e.g. DB needs more RAM)?  â”‚
â”‚          â”‚                                                        â”‚
â”‚     YES â”€â”¼â”€â”€â–º Scale that component vertically.                  â”‚
â”‚          â”‚                                                        â”‚
â”‚     NO   â”‚                                                        â”‚
â”‚          â”‚                                                        â”‚
â”‚          â””â”€â”€â–º Horizontal scaling. Accept the complexity.         â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

---

## 4. Availability â€” how do you stay up when things go wrong?

**Availability** is simply: what percentage of time is your system actually working and reachable by users?

It sounds simple, but getting from "pretty reliable" to "almost never down" is one of the hardest engineering problems.

### The nines â€” what they actually mean

| Availability | Downtime per year | Downtime per month | What it means in practice |
|---|---|---|---|
| 90% | 36.5 days | ~3 days | Awful. Users will leave. |
| 99% | 3.65 days | ~7 hours | Acceptable for hobby projects |
| 99.9% | 8.7 hours | ~44 minutes | "Three nines" â€” industry standard for good services |
| 99.99% | 52 minutes | ~4 minutes | "Four nines" â€” what serious production services aim for |
| 99.999% | 5 minutes | ~26 seconds | "Five nines" â€” telephone networks, banking |
| 99.9999% | 30 seconds | ~3 seconds | Practically unachievable at reasonable cost |

> **52 minutes of downtime per year** sounds like a lot, but achieving "four nines" (99.99%) requires extremely careful engineering. AWS targets this. Most companies settle for 99.9%.

---

### Single Point of Failure â€” the #1 enemy of availability

A **single point of failure (SPOF)** is any component whose failure brings down the entire system.

```
SYSTEM WITH SPOF:                    SAME SYSTEM WITHOUT SPOF:

 Users                                Users
   â”‚                                    â”‚
   â–¼                                    â–¼
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”                     â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  Server  â”‚ â† if this dies,     â”‚ Load Balancerâ”‚ â† if this dies,
â””â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”˜   everything ends   â””â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”˜   deploy a backup LB
     â”‚                              â”‚        â”‚
     â–¼                              â–¼        â–¼
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”                   â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â” â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  Single  â”‚ â† if this dies,   â”‚ Server â”‚ â”‚ Server â”‚
â”‚    DB    â”‚   all data gone   â”‚   A    â”‚ â”‚   B    â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜                   â””â”€â”€â”€â”¬â”€â”€â”€â”€â”˜ â””â”€â”€â”€â”¬â”€â”€â”€â”€â”˜
                                   â”‚           â”‚
                                   â–¼           â–¼
                              â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â” â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
                              â”‚Primary  â”‚ â”‚  Replica  â”‚ â† if Primary dies,
                              â”‚   DB    â”‚ â”‚    DB     â”‚   Replica takes over
                              â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

---

### How availability is achieved in practice

```mermaid
graph TD
    A["High Availability Goals"]

    A --> R["ðŸ” Redundancy\nDuplicate every critical component\nNo single point of failure"]
    A --> F["âš¡ Failover\nAutomatic switch to backup\nwhen primary fails"]
    A --> H["ðŸ’“ Health Checks\nConstantly ping components\nRemove unhealthy ones from rotation"]
    A --> GD["ðŸŒ Geographic Distribution\nRun in multiple regions\nOne region goes down, others take over"]
    A --> GR["ðŸ”„ Graceful Degradation\nIf search is down, show cached results\nDon't fail completely, fail partially"]

    R --> R1["Multiple servers\nbehind load balancer"]
    R --> R2["Database replicas\nin different zones"]
    F --> F1["Automatic: system detects\ndeath and promotes backup\n(seconds)"]
    F --> F2["Manual: human intervenes\n(minutes to hours)"]
    H --> H1["Load balancer sends traffic\nonly to healthy servers"]
    H --> H2["Failed server gets\npulled from rotation"]
```

---

### Graceful Degradation â€” fail smart, not hard

The best systems don't crash completely when something breaks. They degrade gracefully â€” they limp along, serving *some* functionality.

```
âŒ Not graceful:
   Recommendation service is down
   â†’ Entire homepage crashes with 500 error
   â†’ User can't do anything

âœ… Graceful degradation:
   Recommendation service is down
   â†’ Homepage shows generic popular items instead of personalised ones
   â†’ User can still browse, search, buy
   â†’ They don't even notice the recommendation engine is broken
```

> Facebook's core feed might go down, but you can still see your profile. Netflix's recommendations might fail, but you can still search and play movies. This is graceful degradation by design.

---

## 5. CAP Theorem â€” the impossible triangle

This is one of the most famous ideas in all of distributed systems, and it's surprisingly intuitive once you see it clearly.

### What the three letters mean

**C â€” Consistency:** Every read gets the most recent write. If you just updated your profile picture, and your friend loads your profile right now, they see the new one. Not the old one. Always.

**A â€” Availability:** The system always responds. Maybe the data is slightly old, but it never just says "I'm busy" or "I'm down". It always gives you *something*.

**P â€” Partition Tolerance:** The system keeps working even if the network between some of its machines breaks (a "partition" = a communication split between nodes).

---

### The catch

```
                         âœ¨ Consistency âœ¨
                        "Every read gets
                         the latest write"
                               â–²
                              /|\
                             / | \
                            /  |  \
                           /   |   \
                          / CP | CA  \
                         /     |     \
                        /      |      \
                       â–¼       |       â–¼
          Availability â—„â”€â”€â”€â”€â”€â”€â”€â”¤â”€â”€â”€â”€â”€â”€â–º Partition
          "System always        |        Tolerance
           responds"     AP     |    "Works even if
                                |     network splits"
                                â”‚
                      In practice, P is not
                      optional. Networks DO fail.
                      So you pick C or A.
```

**Why P isn't truly optional:**

In any distributed system (multiple machines), network failures happen. A cable gets unplugged. A switch fails. A datacenter loses power. When this happens, your nodes can't talk to each other â€” that's a partition.

Your system has to decide: do you want to **stay consistent** during the partition (refuse to respond until the split heals), or **stay available** (keep responding, even if some nodes have stale data)?

**That's the real choice: CP or AP.**

```
                CP Systems                          AP Systems
         (Consistent + Partition Tolerant)   (Available + Partition Tolerant)

    âœ… Every read = most recent write      âœ… System always responds
    âŒ Might refuse requests during         âŒ Might give slightly old data
       a network partition                    during a partition

    Use for: Bank balances, medical         Use for: Shopping carts, social
    records, booking systems                media feeds, product catalogs
    (wrong data = real harm)               (slightly stale = fine)

    Examples: PostgreSQL, MySQL,           Examples: Cassandra, DynamoDB,
    HBase, Zookeeper                       CouchDB, DNS
```

---

### A concrete example to make it click

Imagine you have a bank and your database is split across two data centres. Suddenly the network cable between them breaks.

```
Data Centre A                          Data Centre B
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”    âŒ BROKEN âŒ    â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  DB Node A     â”‚ â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€  â”‚  DB Node B     â”‚
â”‚  Balance: Â£500 â”‚   network split    â”‚  Balance: Â£500 â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜                    â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
        â”‚                                     â”‚
   User in UK                          User in USA
   withdraws Â£300                      No action
```

Now what?

**If CP:** Node A says "I can't confirm with Node B â€” I won't process this withdrawal. Error 503." The user is unhappy but the data is safe.

**If AP:** Node A says "I'll process it â€” balance is now Â£200." Meanwhile Node B still thinks the balance is Â£500. If the US user also withdraws Â£400 at the same time from Node B... the bank loses money. Bad.

> **Banks are CP.** Your Twitter feed is AP (seeing a tweet 3 seconds late = fine).

---

## 6. Databases â€” choosing the right storage

The database is where all your data lives permanently. Picking the wrong one is one of the most painful mistakes in engineering because changing it later is horrible.

---

### SQL â€” the spreadsheet model

SQL databases store data in **tables** (like spreadsheets), with **rows** (records) and **columns** (fields). The big thing is: tables can be **related** to each other using IDs.

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”     â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚         users                   â”‚     â”‚              orders                  â”‚
â”œâ”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤     â”œâ”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚ id â”‚ name       â”‚ email         â”‚     â”‚ id â”‚ user_id â”‚ amount   â”‚ status    â”‚
â”œâ”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤     â”œâ”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚  1 â”‚ Alice      â”‚ a@gmail.com  â”‚â—„â”€â”€â”€â”€â”‚  1 â”‚    1    â”‚ Â£49.99   â”‚ delivered â”‚
â”‚  2 â”‚ Bob        â”‚ b@gmail.com  â”‚â—„â”€â”€â” â”‚  2 â”‚    1    â”‚ Â£12.00   â”‚ pending   â”‚
â”‚  3 â”‚ Charlie    â”‚ c@gmail.com  â”‚   â””â”€â”‚  3 â”‚    2    â”‚ Â£109.50  â”‚ delivered â”‚
â””â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜     â””â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

You can run a query like: *"Give me all orders over Â£50 for users who signed up before 2024"* â€” joining the two tables together. This is the superpower of SQL.

**The ACID guarantee â€” what makes SQL trustworthy:**

```
A â€” Atomicity:    A transaction either fully succeeds or fully fails.
                  Transfer Â£100 from Alice to Bob:
                  âœ… Alice -Â£100 AND Bob +Â£100 (both happen)
                  âŒ Never: Alice -Â£100 but Bob stays the same (partial)

C â€” Consistency:  After every transaction, all rules are still valid.
                  Can't have an Order with a user_id that doesn't exist.

I â€” Isolation:    Two transactions happening at the same time don't
                  interfere with each other. Like they're queued up.

D â€” Durability:   Once committed, data survives a crash.
                  The database doesn't "forget" after a power cut.
```

**Use SQL for:** Banking, e-commerce, any system where data relationships matter and you cannot afford incorrect data.

Popular choices: PostgreSQL, MySQL, SQLite, Microsoft SQL Server

---

### NoSQL â€” the flexible model

NoSQL doesn't mean "no SQL" â€” it means "not only SQL". There's no single type of NoSQL â€” there are four very different flavours, each built for a specific kind of problem.

```mermaid
graph TD
    Problem["What kind of data are you storing?"]

    Problem --> Q1{"Is it structured\nwith relationships?"}
    Q1 -->|"Yes"| SQL["ðŸ”· SQL\nPostgres, MySQL"]

    Q1 -->|"No"| Q2{"Is it flexible\nJSON-like documents?"}
    Q2 -->|"Yes"| Doc["ðŸ“„ Document DB\nMongoDB, Firestore\n\nEach record is a\nself-contained JSON blob.\nFields can differ per record."]

    Q1 -->|"No"| Q3{"Is it simple\nlookup by key?"}
    Q3 -->|"Yes"| KV["ðŸ”‘ Key-Value\nRedis, DynamoDB\n\nLike a dictionary.\nGive me the thing\nstored under this key."]

    Q1 -->|"No"| Q4{"Is it massive amounts\nof time-series / event data?"}
    Q4 -->|"Yes"| Wide["ðŸ“Š Wide Column\nCassandra, HBase\n\nBuilt for billions of rows.\nEach row can have different\ncolumns. Fast writes."]

    Q1 -->|"No"| Q5{"Is it relationships\nbetween entities?"}
    Q5 -->|"Yes"| Graph["ðŸ•¸ï¸ Graph DB\nNeo4j\n\nNodes = things\nEdges = relationships\nPerfect for social networks"]
```

---

### Document DB â€” a filing cabinet with flexible folders

```
SQL way:                              MongoDB way:
users table + addresses table         One self-contained document:

SELECT u.name, a.city               {
FROM users u                           "name": "Alice",
JOIN addresses a                       "email": "a@gmail.com",
ON u.id = a.user_id                    "addresses": [
WHERE u.id = 1;                           {"city": "London", "primary": true},
                                          {"city": "Manchester"}
                                       ],
                                       "preferences": {
                                          "theme": "dark",
                                          "notifications": false
                                       }
                                    }
```

No joins needed â€” everything about Alice is in one document. Great for reading. Bad if you need to query across many documents in complex ways.

---

### ACID (SQL) vs BASE (NoSQL) â€” the philosophy difference

| | **ACID** (SQL) | **BASE** (NoSQL) |
|---|---|---|
| **Priority** | Correctness above all else | Availability above all else |
| **A / BA** | **A**tomic â€” all or nothing | **B**asically **A**vailable â€” always responds |
| **C / S** | **C**onsistent â€” rules always apply | **S**oft state â€” might be temporarily inconsistent |
| **I / E** | **I**solated â€” transactions don't interfere | **E**ventually consistent â€” will be correct... eventually |
| **D** | **D**urable â€” survives crashes | Depends on config |
| **Analogy** | A super strict accountant who checks every number twice | A fast cashier who sometimes needs to reconcile the till at end of day |
| **Use for** | Money, reservations, medical data | Likes, views, caches, feeds |

---

## 7. Caching â€” stop asking the same question twice

### The basic idea

Every time a user loads your app's homepage, you might be running 10 database queries to build that page. If 100,000 people load the homepage per minute, that's 1,000,000 database queries per minute â€” for the same data.

The cache says: *"I already fetched this. Here's my saved copy. No need to bother the database."*

```
WITHOUT CACHE:                           WITH CACHE:

Request 1: user asks for homepage        Request 1: cache miss â†’ go to DB â†’ cache result
           â†’ query DB (50ms)             Request 2: cache hit â†’ return from memory (1ms)
Request 2: user asks for homepage        Request 3: cache hit â†’ return from memory (1ms)
           â†’ query DB (50ms)             ...
Request 3: user asks for homepage        Request 100K: cache hit â†’ return from memory (1ms)
           â†’ query DB (50ms)
...

After 100K requests:                     After 100K requests:
100,000 DB queries                       1 DB query, 99,999 cache hits
```

---

### The cache hit / miss lifecycle

```mermaid
flowchart TD
    R["Request comes in:\n'Get profile for user 42'"]
    CHK{"Check\nCache"}
    HIT["âœ… Cache HIT\nReturn cached data\n~1ms"]
    MISS["âŒ Cache MISS\nNot in cache"]
    DB["Query Database\n~50ms"]
    STORE["Store result in cache\nwith expiry time TTL"]
    RET["Return result to user"]

    R --> CHK
    CHK -->|"found it"| HIT
    CHK -->|"not found"| MISS
    HIT --> RET
    MISS --> DB
    DB --> STORE
    STORE --> RET
```

---

### Cache eviction â€” what to throw out when the cache is full

Your cache has limited space (it's RAM). When it fills up, something has to go. These are the strategies:

```
LRU â€” Least Recently Used (most common)
â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”
Evict whatever was LAST ACCESSED the longest time ago.

Cache slots (newest to oldest):
[User:99] [User:42] [User:17] [User:05] â† evict this one next
   â”‚          â”‚         â”‚         â”‚
 2s ago    10s ago   45s ago   3min ago

Analogy: Your desk. You throw out the paper you haven't looked at longest.

â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€

LFU â€” Least Frequently Used
â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”
Evict whatever has been accessed the FEWEST total times.

Cache slots with access counts:
[User:42 (980x)] [User:99 (245x)] [User:17 (12x)] [User:05 (3x)] â† evict
    popular          moderate           rare           barely used

Good for: trending content where popular things stay popular

â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€

TTL â€” Time To Live (simplest)
â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”
Every cached item has an expiry timestamp. After that, it's gone.

SET user:42 "..." EX 300   â† expires in 300 seconds (5 minutes)

Analogy: Milk with an expiry date. After that date, throw it out.
Good for: Any data that changes on a predictable schedule.
```

---

### Cache invalidation â€” the hardest problem

> *"There are only two hard things in Computer Science: cache invalidation and naming things."* â€” Phil Karlton

You cached Alice's profile. Alice then updates her profile picture. Now the cache has the old picture and the database has the new one. They're out of sync. How do you fix this?

```
Strategy 1: Write-Through
â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”
When you update DB, also update the cache immediately.

App â†’ Write to DB â”€â”
                   â”œâ”€â”€â–º Cache is always fresh âœ…
App â†’ Write to Cache â”€â”˜

Problem: Every write is 2 operations. Slightly slower writes.
Best for: Systems where reads >> writes and freshness matters.

â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€

Strategy 2: Cache-Aside (most common in practice)
â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”
The app manages the cache manually.
On read: check cache â†’ if miss, read DB â†’ store in cache
On write: write to DB â†’ DELETE the cache entry (invalidate it)
         Next read will be a miss and refetch from DB.

Good for: Flexibility. Works well in practice.
Problem: There's a tiny window where DB has new data but someone
         reads the (not yet deleted) old cache entry.

â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€

Strategy 3: Write-Back (Write-Behind)
â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”
Write to cache immediately. DB gets updated later (async).
Ultra-fast writes. But risky â€” if cache server dies before
DB is updated, you lose data.

Best for: Non-critical high-frequency writes (like view counters).
```

---

### Where caches live in a real system

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                    Caching at every layer                            â”‚
â”‚                                                                      â”‚
â”‚  Browser Cache    â†’ Stores HTML, CSS, JS, images on user's device   â”‚
â”‚       â”‚              No server request needed at all                 â”‚
â”‚       â”‚                                                              â”‚
â”‚  CDN Cache        â†’ Stores static files at edge locations           â”‚
â”‚       â”‚              Closest server to user responds                 â”‚
â”‚       â”‚                                                              â”‚
â”‚  App Server Cache â†’ Process-level in-memory cache (simple maps)     â”‚
â”‚       â”‚              Fastest but lost when server restarts           â”‚
â”‚       â”‚                                                              â”‚
â”‚  Distributed Cache â†’ Redis / Memcached shared across all servers    â”‚
â”‚       â”‚              The most common production setup                â”‚
â”‚       â”‚              Survives server restarts                        â”‚
â”‚       â”‚                                                              â”‚
â”‚  DB Query Cache   â†’ Database stores results of recent queries        â”‚
â”‚                      Automatically invalidated on data change        â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

---

## 8. Load Balancers â€” the traffic cops

### Why you need one

Imagine a highway suddenly getting 10x the traffic it was designed for. Without any management, every lane piles up and nothing moves. A traffic management system opens new lanes and routes cars intelligently.

A load balancer is exactly that for your servers.

```
WITHOUT LOAD BALANCER:              WITH LOAD BALANCER:

  1000 requests/sec                   1000 requests/sec
         â”‚                                    â”‚
         â–¼                            â”Œâ”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”                     â”‚    Load Balancer     â”‚
  â”‚  Server A   â”‚ â† getting hammered  â”‚  "Server A has 320   â”‚
  â”‚  CPU: 99%   â”‚                     â”‚   requests. Send     â”‚
  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜                     â”‚   next one to B."    â”‚
  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”                     â””â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”˜
  â”‚  Server B   â”‚ â† sitting idle          â”‚      â”‚      â”‚
  â”‚  CPU: 1%    â”‚                         â–¼      â–¼      â–¼
  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜                    Server A  Server B  Server C
  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”                    CPU:33%   CPU:33%   CPU:33%
  â”‚  Server C   â”‚ â† sitting idle
  â”‚  CPU: 1%    â”‚
  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

---

### How a load balancer decides where to send traffic

```mermaid
flowchart TD
    T["Incoming Request"]
    ALG{"Which algorithm\nis configured?"}

    T --> ALG

    ALG -->|"Round Robin"| RR["Send to next server\nin a circular list\n1â†’2â†’3â†’1â†’2â†’3...\n\nBest for: similar servers,\nsimilar request times"]

    ALG -->|"Weighted\nRound Robin"| WRR["Server A (powerful): 60%\nServer B (normal):  30%\nServer C (small):   10%\n\nBest for: mixed hardware"]

    ALG -->|"Least\nConnections"| LC["Check which server has\nfewest active connections\nright now. Send there.\n\nBest for: long-running\nwebsocket connections"]

    ALG -->|"IP Hash"| IPH["Hash the user's IP address\nSame user â†’ always same server\n\nBest for: when server needs\nto remember session state"]

    ALG -->|"Random"| RAND["Pick a random server\n\nSurprisingly effective.\nGood for: simple cases"]
```

---

### Health checks â€” how the load balancer knows a server is dead

Every few seconds, the load balancer pings each server: *"Are you alive?"*

```
Every 5 seconds:
Load Balancer â†’ "GET /health" â†’ Server A â†’ "200 OK" âœ… Keep in rotation
Load Balancer â†’ "GET /health" â†’ Server B â†’ "200 OK" âœ… Keep in rotation
Load Balancer â†’ "GET /health" â†’ Server C â†’ â³ No response after 3s âŒ

Load Balancer: "Server C looks dead. Stop sending it traffic."

All traffic now split between A and B.
Someone alerts the on-call engineer about Server C.
```

When Server C recovers, it gets added back automatically.

---

### Layer 4 vs Layer 7 Load Balancers

There are two types, and understanding the difference matters:

```
Layer 4 â€” Network Load Balancer:
â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”
Operates at TCP/UDP level.
Routes packets based only on: IP address and port number.
Does NOT look inside the request at all.

Like a post office that routes packages by city/zip code only.
Very fast. Can't make smart decisions about content.

Layer 7 â€” Application Load Balancer:
â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”
Operates at HTTP level.
Can look INSIDE the request: URL, headers, cookies, body.
Can make smart routing decisions:

  /api/*       â†’ route to API servers
  /images/*    â†’ route to image servers
  /websocket   â†’ route to websocket servers
  Cookie: beta_user=true â†’ route to staging servers

Like a smart receptionist who reads the letter and sends it to
the right department.
Slower than L4, but much smarter.
```

---

## 9. CDN â€” the world is big, servers are slow

### The speed of light problem

No matter how fast your servers are, there's a physical limit â€” light (and therefore data) travels at about 200,000 km/second through fibre optic cable.

```
Your server is in London.
A user in Sydney wants your homepage image (1MB).

Distance London â†’ Sydney: ~16,800 km
Round trip: ~33,600 km
At speed of light in fibre: 33,600 / 200,000 = 0.168 seconds

Just the physical travel time = 168 milliseconds.
Before your server even processes anything.

With a CDN server IN Sydney:
Distance Sydney â†’ Sydney CDN: ~5 km
Round trip: ~10 km
Travel time: 0.05ms

168ms â†’ 0.05ms. That's 3,360x faster just from proximity.
```

---

### How a CDN works

```mermaid
sequenceDiagram
    participant U as ðŸ‘¤ User in Tokyo
    participant CDN as ðŸ“¦ CDN Edge Server (Tokyo)
    participant O as ðŸ¢ Origin Server (New York)

    Note over U,O: First time anyone in Tokyo requests this image
    U->>CDN: "Give me /images/logo.png"
    CDN->>O: "I don't have it yet. Origin, send it to me."
    O-->>CDN: "Here's logo.png" (sent once, 200ms)
    CDN->>CDN: Store logo.png locally
    CDN-->>U: "Here's logo.png" (served from Tokyo, fast!)

    Note over U,O: Every subsequent request from anywhere in Tokyo
    U->>CDN: "Give me /images/logo.png"
    CDN-->>U: "Here it is!" (from local storage, ~5ms)

    Note over U,O: Origin never gets hit again until CDN cache expires
```

---

### Push CDN vs Pull CDN

```
PULL CDN (most common):
â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”
CDN fetches content from your origin ON DEMAND when first requested.
You don't have to do anything special.
CDN "pulls" content lazily as users request it.

Workflow: User requests â†’ CDN checks storage â†’ miss â†’ CDN fetches
          from origin â†’ CDN stores it â†’ CDN serves user.

Best for: Websites where you can't predict which content gets popular.

PUSH CDN:
â”â”â”â”â”â”â”â”â”
YOU proactively upload content to the CDN before users ask for it.
CDN always has it ready. Zero origin requests.

Workflow: You deploy new content â†’ you push it to CDN via API â†’
          CDN distributes to all edge locations â†’ users hit CDN directly.

Best for: Scheduled video releases (Netflix new episodes), software
          updates â€” you know exactly when it goes live.
```

---

## 10. Message Queues â€” don't do things right now

### The real problem they solve

Without queues, your system is **synchronous** â€” the user has to wait for EVERYTHING to finish before getting a response. Tasks are chained together.

```
User clicks "Place Order":
â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
1. Save order to DB             50ms    â† user waiting
2. Check inventory              30ms    â† still waiting
3. Charge credit card          200ms    â† still waiting
4. Send confirmation email     150ms    â† still waiting
5. Update inventory count       30ms    â† still waiting
6. Notify warehouse              40ms   â† still waiting
â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
Total wait: 500ms of user staring at a spinner.

And if ANY step fails, the user sees an error.
```

With a queue, you only do the essential and fast parts synchronously. Everything else gets queued:

```
User clicks "Place Order":
â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
1. Save order to DB             50ms    â† user waiting
2. Charge credit card          200ms    â† still waiting
3. Put these jobs on queue:      1ms    â† submit and forget
     â”Œâ”€ "send confirmation email"
     â”œâ”€ "update inventory"
     â””â”€ "notify warehouse"
4. Return "Order confirmed!" â† user is happy in 251ms
â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€

Meanwhile, workers process the queue jobs at their own pace:
Email worker:     sends email       (might take 2 seconds, user doesn't care)
Inventory worker: updates count     (might take 1 second)
Warehouse worker: notifies system   (might take 500ms)
```

---

### The queue in detail

```mermaid
graph LR
    subgraph Producers
        P1["ðŸ“± Web Server"]
        P2["ðŸ“± Mobile API"]
        P3["ðŸ”„ Cron Job"]
    end

    subgraph Queue
        Q["ðŸ“¬ Message Queue\n\nJob 1: send-email\nJob 2: resize-image\nJob 3: send-email\nJob 4: update-feed\nJob 5: send-push\n..."]
    end

    subgraph Workers - Consumers
        W1["âš™ï¸ Email Worker\n(takes email jobs)"]
        W2["âš™ï¸ Image Worker\n(takes image jobs)"]
        W3["âš™ï¸ Feed Worker\n(takes feed jobs)"]
    end

    P1 -->|"publish job"| Q
    P2 -->|"publish job"| Q
    P3 -->|"publish job"| Q

    Q -->|"delivers job"| W1
    Q -->|"delivers job"| W2
    Q -->|"delivers job"| W3
```

**The magic is decoupling:** Producers don't know or care who processes their jobs. Workers don't know or care who created the jobs. Each side can scale independently. If emails slow down, add more email workers. If image processing is fast, remove image workers.

---

### What happens when a worker fails mid-job?

This is where queues are genuinely clever. Most queues use **acknowledgement (ACK)**:

```
1. Worker picks up "send email" job
2. Worker starts processing it
3. Server crashes mid-way â€” email never sent
4. Queue notices: "I gave this job 30 seconds ago and got no ACK. 
   Putting it back in the queue."
5. Another worker picks it up and completes it âœ…

Without a queue:
  Job was being processed in memory â†’ server crashes â†’ job gone forever
  User never gets their email. No retry. Silent failure.
```

---

### Dead Letter Queue (DLQ) â€” where jobs go to die

Some jobs fail repeatedly no matter how many times you retry. Maybe the email address is invalid. Maybe the data is malformed. After N retries, they go to the DLQ.

```
Job fails â†’ retry â†’ fail â†’ retry â†’ fail â†’ retry â†’ fail
After 3 retries: â†’ Dead Letter Queue

Operations team can inspect DLQ:
  "Why are these 47 jobs failing?"
  "Ah, the email service API changed format. Let's fix it and replay."
```

---

### Kafka vs RabbitMQ vs SQS

| | Kafka | RabbitMQ | AWS SQS |
|---|---|---|---|
| **Mental model** | Append-only log. Messages are retained for days/weeks. Multiple consumers can read the same messages. | Traditional queue. Message is deleted once a consumer processes it. | Managed queue. Fully hosted by AWS. |
| **Speed** | Millions of msg/sec | Hundreds of thousands/sec | Moderate, but managed |
| **Replay** | âœ… Yes â€” rewind to any point | âŒ No â€” gone once consumed | âŒ No (standard queue) |
| **Use for** | Event streaming, audit logs, activity feeds | Task queues, async jobs | AWS-native async processing |
| **Analogy** | A newspaper archive â€” every edition ever printed, you can go back and re-read any of them | A sticky note â€” once acted on, you throw it away | A managed sticky note service |

---

## 11. API Gateway â€” the front door

### Why you need one

Imagine a hospital. Every department â€” cardiology, neurology, pharmacy, radiology â€” is a separate service. Patients don't walk directly into each department. They go to **reception** first. Reception checks who they are, decides where to send them, and logs the visit.

The API Gateway is reception.

```
WITHOUT GATEWAY:                    WITH GATEWAY:

Mobile App â†’ user-service:3001      Mobile App â”€â”€â–º API Gateway
Mobile App â†’ order-service:3002                        â”‚
Mobile App â†’ payment-service:3003      Checks:         â”œâ”€â”€â–º user-service
Mobile App â†’ search-service:3004       - Auth?          â”œâ”€â”€â–º order-service
                                       - Rate limit?     â”œâ”€â”€â–º payment-service
Browser â†’ user-service:3001            - Which service? â”œâ”€â”€â–º search-service
Browser â†’ order-service:3002           - Log the call   â””â”€â”€â–º ...
...

App needs to know every service's address.    App only knows one address.
Every service implements auth independently.  Auth is centralised.
Every service implements rate limiting.       Rate limiting is centralised.
```

---

### What the API Gateway does step by step

```mermaid
flowchart TD
    REQ["Incoming Request\nPOST /orders\nAuthorization: Bearer xyz123"]

    REQ --> AUTH{"1. Authenticate\nIs this token valid?"}
    AUTH -->|"invalid token"| ERR1["401 Unauthorized"]
    AUTH -->|"valid token"| RATE{"2. Rate Limit Check\nHas this user exceeded\ntheir request quota?"}
    RATE -->|"quota exceeded"| ERR2["429 Too Many Requests"]
    RATE -->|"within limits"| LOG["3. Log the request\n(timestamp, user, endpoint)"]
    LOG --> SSL["4. SSL Termination\nDecrypt HTTPS here.\nForward as HTTP internally."]
    SSL --> ROUTE{"5. Route to service\nWhich service handles\nPOST /orders?"}
    ROUTE --> SVC["Order Service"]
    SVC --> RESP["Response from service"]
    RESP --> TRANS["6. Transform if needed\n(change format, headers)"]
    TRANS --> USER["Return to Client"]
```

---

## 12. Rate Limiting â€” slowing down the greedy

### Why it matters

Without rate limiting, consider:

- A competitor scrapes your entire product database overnight
- A bot tries 10 million password combinations on your login endpoint
- A misbehaving client has an infinite loop hitting your API
- A DDoS attack floods you with traffic

Rate limiting prevents all of these.

---

### The Token Bucket â€” how it actually works

This is the most intuitive algorithm and the most widely used.

```
Imagine a bucket that:
  - Has a maximum capacity of 100 tokens
  - Fills up at a rate of 10 tokens per second
  - Each API request uses 1 token
  - If the bucket is empty, the request is rejected (429 error)

Timeline:
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ Second 0:  Bucket = 100 tokens (full)                            â”‚
â”‚ Second 1:  5 requests come in  â†’ Bucket = 100 - 5 + 10 = 105... â”‚
â”‚            ... capped at 100   â†’ Bucket = 100                    â”‚
â”‚ Second 2:  80 requests (burst) â†’ Bucket = 100 - 80 + 10 = 30    â”‚
â”‚ Second 3:  80 requests again   â†’ Bucket = 30 - 80 + 10 = -40    â”‚
â”‚            Bucket can't go negative. 40 requests get 429 âŒ      â”‚
â”‚            10 go through âœ…                                       â”‚
â”‚ Second 4:  Bucket refills: 10 new tokens. Bucket = 20            â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜

Benefit of Token Bucket: allows SHORT bursts (up to bucket capacity)
but sustains the long-term average rate.
A legitimate user who does a sudden burst of activity isn't blocked
immediately â€” they use their saved-up tokens.
```

---

### The algorithms compared

```
Fixed Window:
â”â”â”â”â”â”â”â”â”â”â”â”
Allow 100 requests per minute.
Window resets every 60 seconds.

|---60s window---|---60s window---|
0              60              120

Problem: User sends 100 at second 59, then 100 at second 61.
That's 200 in 2 seconds. Both windows say "fine!" âŒ

â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€

Sliding Window:
â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”
Look back exactly 60 seconds from RIGHT NOW, not from a fixed reset.

At second 61, user wants request 101:
  Count requests from second 1 to 61: all 100 from window 1 gone âœ…
  Correctly handled.

More accurate but needs more memory.

â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€

Leaky Bucket:
â”â”â”â”â”â”â”â”â”â”â”â”â”
Requests go IN at any rate.
Requests come OUT at a fixed, constant rate.

Like a bucket with a small hole in the bottom.
Water (requests) pours in fast. Drains at steady pace.
If you pour too fast, water overflows (requests dropped).

Guarantees a perfectly smooth output rate.
Bad for bursty-but-legitimate traffic.
```

---

## 13. Database Sharding â€” cutting the data pie

### When do you even need this?

Most apps never need sharding. You need it only when your data grows so large that a single database machine can't hold it or query it fast enough.

Signs you might need sharding:
- Your database table has billions of rows and queries take seconds
- A single DB server runs out of disk space
- Write throughput exceeds what one machine can handle (~many thousands/sec)

---

### What sharding actually is

You split your data across multiple database machines called **shards**. Each shard holds a subset of the total data.

```
WITHOUT SHARDING:                    WITH SHARDING:

One database                         Shard 1       Shard 2       Shard 3
storing all                          â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”   â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”   â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”
1 billion users:                     â”‚Users    â”‚   â”‚Users    â”‚   â”‚Users    â”‚
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”                      â”‚ID 1-333Mâ”‚   â”‚ID 333M- â”‚   â”‚ID 666M- â”‚
â”‚ ALL 1B rows â”‚                      â”‚         â”‚   â”‚666M     â”‚   â”‚1B       â”‚
â”‚ crushing    â”‚                      â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜   â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜   â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
â”‚ one machine â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜                      Each shard is a separate machine.
                                     Each holds ~333M rows.
```

---

### Shard key â€” the decision that haunts you

The shard key is the field you use to decide which shard a piece of data goes to. Choose wrong and you create "hot spots" â€” one shard getting way more traffic than others.

```mermaid
graph TD
    Query["Query for user_id = 7,382,194"]
    SK{"Shard Key:\nuser_id % 3"}

    Query --> SK

    SK -->|"7,382,194 % 3 = 1"| S1["ðŸ—„ï¸ Shard 1"]
    SK -->|"7,382,194 % 3 = 2"| S2["ðŸ—„ï¸ Shard 2"]
    SK -->|"7,382,194 % 3 = 0"| S3["ðŸ—„ï¸ Shard 3"]

    S1 --> FOUND["Found! Return user data"]
```

**Hot spot example â€” why shard key choice matters:**

```
BAD shard key: created_date (range-based: Jan-Apr â†’ Shard 1, etc.)

January:  10,000 users signing up â†’ all hit Shard 1
Feb-Apr:  10,000 more             â†’ all still hit Shard 1
May-Aug:  10,000 more             â†’ all hit Shard 2
Current month: ALL activity       â†’ all hit whichever shard is current

Result: Current shard is on fire. All others are idle.
This is called a HOT SPOT. âŒ

GOOD shard key: hash(user_id) % number_of_shards

Every user_id hashes to a pseudorandom shard.
Traffic is distributed evenly.
No hot spots. âœ…
```

---

### The dark side of sharding â€” why to avoid it until you must

```
Problem 1: Cross-shard queries
â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”
"Find all users who bought product X"
â†’ Has to query ALL shards and merge results
â†’ Slow, complex, expensive

Problem 2: Cross-shard transactions
â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”
User A (Shard 1) transfers money to User B (Shard 3)
â†’ Can't do an ACID transaction across shards
â†’ Need complex distributed transaction protocols (nightmare)

Problem 3: Resharding
â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”
You have 3 shards. You add a 4th.
Now hash(user_id) % 4 â‰  hash(user_id) % 3
â†’ ALL existing data is on the wrong shard
â†’ You have to migrate everything while the app is live
â†’ Engineering hell

Problem 4: Joins don't work
â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”
SQL JOIN between two tables that live on different shards = not possible
You have to do it in application logic.
```

> **The rule:** Don't shard. Scale vertically first. Add read replicas. Optimise your queries. Only shard as a last resort.

---

## 14. Replication â€” keeping backups alive

### Sharding vs Replication â€” what's the difference?

```
Sharding:      Split data across machines (each holds different data)
               Goal: Handle more data / more writes

Replication:   Copy SAME data across machines
               Goal: Handle more reads / survive failures
```

---

### How Primary-Replica replication works

```mermaid
graph LR
    APP["Application"]

    APP -->|"ALL writes go here"| P["ðŸ—„ï¸ Primary DB\n(Read + Write)"]
    P -->|"replicates changes"| R1["ðŸ—„ï¸ Replica 1\n(Read only)"]
    P -->|"replicates changes"| R2["ðŸ—„ï¸ Replica 2\n(Read only)"]
    P -->|"replicates changes"| R3["ðŸ—„ï¸ Replica 3\n(Read only)"]

    APP -->|"reads can go to any replica"| R1
    APP -->|"reads can go to any replica"| R2
    APP -->|"reads can go to any replica"| R3
```

Why separate reads and writes?
Most apps have far more reads than writes (e.g. 90% reads, 10% writes). Replicas handle all the reading, freeing up the primary to focus purely on writes. This is a massive performance win.

---

### Synchronous vs Asynchronous Replication

```
Synchronous Replication:
â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”
1. App writes to Primary
2. Primary writes to disk
3. Primary sends data to Replica
4. Replica writes to disk and confirms
5. Primary says "Done!" to App

     App â”€â”€â–º Primary â”€â”€â–º Replica â”€â”€â–º confirms
                  â—„â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
              Only responds after replica confirms

âœ… Zero data loss. Replica always up to date.
âŒ Every write takes longer (waiting for replica ACK).
âŒ If replica is slow or down, primary stalls.

Asynchronous Replication:
â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”
1. App writes to Primary
2. Primary writes to disk
3. Primary immediately says "Done!" to App
4. Primary sends data to Replica in background (best effort)

     App â”€â”€â–º Primary â”€â”€â–º confirms immediately
                  â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â–º
             Sends to replica later, asynchronously

âœ… Fast writes. Primary not slowed by replica.
âŒ Tiny window where replica is behind (called "replication lag").
âŒ If Primary crashes before replicating, last few writes are lost.
```

---

### What happens when the Primary dies?

This is called **failover**. The system needs to promote one of the replicas to become the new primary.

```
Normal state:                         After Primary dies:
Primary â”€â”€â–º Replica 1                 âŒ Primary is dead
        â”€â”€â–º Replica 2                 Replica 1 nominated as new Primary
        â”€â”€â–º Replica 3                 Replica 1 â”€â”€â–º Replica 2
                                                â”€â”€â–º Replica 3

Automatic failover (e.g. in AWS RDS):
  - Primary stops responding
  - Health checks detect it within seconds
  - A replica is automatically promoted
  - DNS is updated to point to new Primary
  - Total downtime: 30-60 seconds

Manual failover:
  - Someone gets paged at 2am
  - They log in, assess, decide which replica to promote
  - They manually run the promotion commands
  - Total downtime: minutes to hours
```

---

## 15. Microservices vs Monolith â€” one big thing or many small things?

This is one of the most debated topics in software engineering, and the honest answer is: **both have their place**.

---

### The Monolith â€” start here

A monolith is one codebase, deployed as one unit, talking to one database.

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                     Monolith                                     â”‚
â”‚                                                                  â”‚
â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”          â”‚
â”‚  â”‚ User Module  â”‚  â”‚ Order Module â”‚  â”‚Search Module â”‚          â”‚
â”‚  â””â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”˜  â””â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”˜  â””â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”˜          â”‚
â”‚         â”‚                 â”‚                  â”‚                   â”‚
â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”  â”Œâ”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”  â”Œâ”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”          â”‚
â”‚  â”‚ Auth Module  â”‚  â”‚Payment Moduleâ”‚  â”‚ Email Module â”‚          â”‚
â”‚  â””â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”˜  â””â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”˜  â””â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”˜          â”‚
â”‚         â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜                  â”‚
â”‚                            â”‚                                     â”‚
â”‚               â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”                       â”‚
â”‚               â”‚      One Database       â”‚                       â”‚
â”‚               â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜                       â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
                      One deployment unit
                      One codebase
                      One repository
```

**It's actually great when you're small:**
- Easy to run locally â€” just one `npm start`
- Easy to understand â€” everything is in one place
- Easy to debug â€” no network calls between modules, just function calls
- Works fine at small to medium scale

**It falls apart when you grow:**
- 50 engineers all changing one codebase â†’ constant merge conflicts
- One module's bug can crash the entire app
- "We need to scale the search feature" â†’ you have to scale the *entire* monolith
- Deploy is all or nothing â€” a small fix requires deploying everything

---

### Microservices â€” split by business domain

Each service is its own independent app: separate codebase, separate database, separate deployment, separate team.

```mermaid
graph TB
    GW["ðŸšª API Gateway"]

    GW --> US["ðŸ‘¤ User Service\n\nOwns user data\nNode.js + PostgreSQL\nTeam A"]
    GW --> OS["ðŸ“¦ Order Service\n\nOwns order data\nPython + MongoDB\nTeam B"]
    GW --> PS["ðŸ’³ Payment Service\n\nOwns payment data\nJava + MySQL\nTeam C"]
    GW --> NS["ðŸ”” Notification Service\n\nSends emails/push\nGo + Redis\nTeam D"]
    GW --> SS["ðŸ” Search Service\n\nSearch index\nElasticsearch\nTeam E"]

    OS -->|"'order placed' event"| MQ["ðŸ“¬ Message Queue"]
    MQ --> PS
    MQ --> NS
    PS -->|"'payment done' event"| MQ
    MQ --> NS
```

**The benefits:**
- Each service deploys independently â€” fix a bug in payments without touching search
- Each service scales independently â€” payments getting slow? Scale only that
- Teams own their service fully â€” no stepping on each other
- Technology freedom â€” use the best tool for each job

**The cost:**
- Network calls between services add latency and failure risk  
- A request might touch 5 services â€” debugging requires tracing across all 5  
- Data consistency across services is hard (no single DB to ACID across)  
- Much more infrastructure to manage (50 services = 50 CI/CD pipelines)  

---

### The honest guide to choosing

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                                                                  â”‚
â”‚  Team size < 15 engineers?                                       â”‚
â”‚          â”‚                                                       â”‚
â”‚  YES â”€â”€â”€â”€â”¤â”€â”€â–º Start with monolith. 100%.                        â”‚
â”‚          â”‚    Microservices will slow you down, not help you.    â”‚
â”‚          â”‚                                                       â”‚
â”‚  Team 15-50?                                                     â”‚
â”‚          â”‚                                                       â”‚
â”‚  YES â”€â”€â”€â”€â”¤â”€â”€â–º Modular monolith.                                  â”‚
â”‚          â”‚    Well-structured monolith with clear module          â”‚
â”‚          â”‚    boundaries. When those boundaries hurt,            â”‚
â”‚          â”‚    extract one service at a time.                     â”‚
â”‚          â”‚                                                       â”‚
â”‚  Team 50+ and specific pain?                                     â”‚
â”‚          â”‚                                                       â”‚
â”‚  YES â”€â”€â”€â”€â”´â”€â”€â–º Extract services for the parts that are:          â”‚
â”‚               - Scaling differently from the rest               â”‚
â”‚               - Deployed 10x more often than the rest           â”‚
â”‚               - Owned by a dedicated team                       â”‚
â”‚               - Genuinely independent                           â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

> Amazon famously runs hundreds of microservices. They also started as a monolith and grew into it over years. They didn't start with microservices on day one.

---

## 16. Real World Examples â€” how the big apps actually work

### How Twitter / X handles 500 million tweets per day

The core problem is the **fan-out problem**: When Elon Musk (200M followers) tweets, 200 million people's feeds potentially need updating.

```mermaid
graph LR
    T["âœï¸ Tweet created\nby celebrity"]
    TS["Tweet Service\nSaves tweet to DB\nAssigns tweet ID"]
    FS["Fan-out Service\nLooks up all followers"]
    Q["ðŸ“¬ Message Queue\n'Write tweet:123\nto user:X timeline'\n\n200M messages\nqueued up"]
    TW["âš™ï¸ Timeline Workers\n(many parallel)\nWrite to each\nfollower's cache"]
    TC["âš¡ Timeline Cache\n(one per user)\nPre-computed feed"]
    R["Read Service\nAssembles timeline\nfrom cache"]
    F["ðŸ“± Followers\nopen Twitter"]

    T --> TS
    TS --> FS
    FS --> Q
    Q --> TW
    TW --> TC
    F --> R
    R --> TC
```

**The tricky design decision â€” Push vs Pull:**

```
Push (Fan-out on write):
  When tweet is posted â†’ immediately write to every follower's timeline cache
  âœ… Reading feed is instant (just read cache)
  âŒ Celebrity with 200M followers = 200M cache writes per tweet
  âŒ Slow to post if you have many followers

Pull (Fan-out on read):
  When tweet is posted â†’ just save the tweet
  When user opens app â†’ compute their feed on the fly
  âœ… Posting is instant
  âŒ Opening feed requires checking every person you follow â†’ slow

Twitter's actual solution: HYBRID
  Regular users (< ~10K followers) â†’ Push. Pre-compute their fans' timelines.
  Celebrities (> ~10K followers) â†’ Pull. Too expensive to fan-out.
  When you open your feed: pre-computed timeline + fetch celebrity tweets live, merge.
```

---

### How Netflix serves video to 260 million subscribers

The hard part: video files are enormous (4K film = 100+ GB). You can't serve them from one server. You can't even serve them from one country.

```
What happens when you press Play on "Stranger Things":

1. Browser â†’ API Gateway
   Authenticated? What plan are you on? What quality can you get?

2. API Gateway â†’ Playback Service
   "Here is the manifest file: a list of video chunks at each quality"
   
3. Your Netflix app reads the manifest:
   "There are 3000 chunks. Each 2 seconds of video. 
    Available in 4K, 1080p, 720p, 480p."

4. App checks your bandwidth: "You're getting 20 Mbps"
   Picks: 1080p quality. Requests first 10 chunks.

5. Chunks are fetched from CDN server CLOSEST to you (Open Connect)
   Netflix has their own CDN nodes installed in ISPs worldwide.
   Your ISP has Netflix content cached locally.

6. Every 30 seconds: app measures actual throughput
   "Bandwidth dropped to 5 Mbps"  â†’ switches to 720p automatically
   "Bandwidth jumped to 50 Mbps"  â†’ switches to 4K automatically

The result: no buffering, automatic quality adjustment.
```

```
Netflix encoding pipeline (before a show even goes live):
â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”
Raw video file (one episode)
         â”‚
         â–¼
Encoding farm (thousands of servers running parallel)
  - 4K HDR (60 Mbps)
  - 4K SDR (20 Mbps)
  - 1080p (8 Mbps)
  - 720p  (4 Mbps)
  - 480p  (2 Mbps)
  - 360p  (1 Mbps)
  Ã— multiple audio languages
  Ã— multiple subtitle tracks
  Ã— multiple codecs (H.264, H.265, AV1)
â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•
100+ encoded versions per episode!

All stored in S3 (object storage)
Then pushed to CDN edge nodes worldwide
```

---

### How Uber matches 5 million rides per day

The hard part: both the rider and driver are moving. Location data from millions of drivers pours in every 4 seconds.

```mermaid
sequenceDiagram
    participant R as ðŸ“± Rider App
    participant GW as API Gateway
    participant LS as Location Service
    participant MS as Matching Service
    participant DA as ðŸ“± Driver App

    Note over DA,LS: Every 4 seconds, constantly:
    DA->>LS: My location: 51.507Â°N, 0.127Â°W

    Note over R,GW: Rider requests a ride:
    R->>GW: "I need a ride. I'm at [location]"
    GW->>MS: Find available drivers near [location]
    MS->>LS: "Give me all drivers within 2km of [location]"
    LS-->>MS: [Driver A: 0.4km, Driver B: 0.9km, Driver C: 1.5km, ...]
    MS->>MS: Score drivers (distance + rating + car type)
    MS->>DA: "New ride request from rider at [location]"
    DA-->>MS: "Accepted"
    MS-->>GW: Matched with Driver A
    GW-->>R: "Driver arriving in 4 minutes. Blue Toyota, XYZ123"

    Note over R,DA: Ride in progress:
    loop Every 4 seconds
        DA->>LS: Updated location
        LS->>R: Show driver location on map
    end
```

**How the geospatial search works:**

The location service needs to answer *"who is within 2km of this point?"* millions of times per second. Standard databases are terrible at this. Uber uses a technique called **geohashing**:

```
Geohash: divide the world into a grid
  Each grid cell has a string code.
  Nearby locations have similar code prefixes.

  Location: 51.507Â°N, 0.127Â°W
  Geohash:  gcpvh3  (6-character code)

  Finding nearby drivers:
  â†’ Find all drivers whose geohash starts with "gcpvh"
  â†’ This is a simple string prefix search â€” database index handles it instantly
  â†’ All matching drivers are within ~2.4km of the requested location

  This turns "find nearby things" into "find things with similar string"
  which any database can do extremely fast.
```

---

## 17. The System Design Interview Playbook

System design interviews test whether you can think through building a real system â€” not whether you memorise every detail. They want to see your thought process.

### The 5-step framework

```mermaid
flowchart LR
    S1["Step 1\nðŸŽ¯ CLARIFY\n5 minutes\n\nAsk questions before\nyou start designing"]
    S2["Step 2\nðŸ“ ESTIMATE\n3 minutes\n\nBack-of-envelope math\nto understand scale"]
    S3["Step 3\nðŸ—ºï¸ HIGH LEVEL\n10 minutes\n\nDraw the boxes and arrows.\nWalk through a full request."]
    S4["Step 4\nðŸ”¬ DEEP DIVE\n15 minutes\n\nGo deep on the hard parts."]
    S5["Step 5\nâš ï¸ EDGE CASES\n5 minutes\n\nWhat breaks?\nHow do you recover?"]

    S1 --> S2 --> S3 --> S4 --> S5
```

---

### Step 1 â€” The questions you MUST ask before drawing anything

*Never start designing without clarifying scope. Interviewers often leave requirements vague on purpose to see if you clarify.*

```
Scope:
  "What are the top 3 features we're designing for?"
  "Are we designing the whole system or one specific part?"

Scale:
  "How many daily active users?"
  "How many requests per second at peak?"
  "How much data are we storing? GB? TB? PB?"

Constraints:
  "Is this read-heavy or write-heavy?"
  "Does it need to be globally available?"
  "What's the latency requirement? Sub-100ms?"

Consistency:
  "Is it okay to show slightly stale data?"
  "Example: is it okay if a user sees their new profile pic 
   with 5 seconds delay, or must it be instant?"
```

---

### Step 2 â€” Back-of-envelope math

You don't need exact numbers. You need order-of-magnitude estimates to guide decisions.

```
Example: "Design Instagram"

Given: 1 billion users, 100M daily active, 50M new photos/day

Requests per second:
  100M DAU, average 10 page loads/day = 1B requests/day
  1B / 86,400 seconds = ~11,500 req/sec
  Peak (10x): ~115,000 req/sec  â†’  Need serious infrastructure

Storage for photos:
  50M new photos/day Ã— 2MB average = 100TB/day
  Per year: 36.5 PB  â†’  Need distributed object storage (S3)
  Can't fit on one machine. Need CDN for delivery.

Database reads:
  Photos are viewed far more than uploaded.
  Assume 100x read:write ratio.
  50M uploads â†’ 5B reads/day â†’ Cache the popular ones!
```

---

### Step 3 â€” The high level diagram

Draw this first, then walk through it:

```
1. Client (browser/mobile)
2. DNS â†’ CDN (for static assets)
3. Load Balancer
4. API Gateway (auth, routing, rate limiting)
5. Application Servers (your main logic)
6. Cache (Redis) â€” in front of DB
7. Primary Database
8. Replicas (for read scale)
9. Object Storage (images/videos)
10. Message Queue (background jobs)
11. Worker Servers
```

---

### Step 4 â€” Deep dive areas to focus on

These are the areas most often probed in interviews:

```
Database Design:
  - What tables/collections? What columns?
  - What are the access patterns? (mostly reads? writes? both?)
  - What needs indexes?

API Design:
  - What endpoints? POST /tweets, GET /feed/:userId
  - What does each accept and return?

The Hard Problem:
  Every system design question has ONE hard problem.
  Find it and go deep.

  Twitter â†’ fan-out to millions of followers
  Uber    â†’ real-time geospatial matching
  YouTube â†’ video encoding and distribution
  WhatsApp â†’ message delivery guarantees
  Pastebin â†’ unique URL generation at scale
```

---

### Quick problem â†’ solution reference

| You hear this problem | Reach for this |
|---|---|
| "Server is overwhelmed" | Horizontal scaling + Load balancer |
| "Database queries are slow" | Add Redis cache in front of it |
| "Database can't hold all the data" | Sharding |
| "One machine dying takes everything down" | Replication + failover, redundancy |
| "Users in other countries have high latency" | CDN + edge servers / multi-region |
| "Users wait too long for heavy operations" | Message queue + async workers |
| "Same data fetched millions of times" | Cache it (browser, CDN, or Redis) |
| "Managing 50 services is chaos" | API Gateway + service mesh |
| "One client is abusing the API" | Rate limiting |
| "Debugging across services is impossible" | Distributed tracing (Jaeger, Zipkin) |
| "Need to see what's happening in real time" | Centralised logging (ELK stack) |
| "Write throughput maxed out" | CQRS + event sourcing, or sharding |

---

## 18. Key Numbers Every Designer Should Know

These numbers come up constantly. You don't need to memorise exact values, just order of magnitude.

```
STORAGE:
â”â”â”â”â”â”â”
1 byte    = one character
1 KB      = 1,000 bytes    â‰ˆ a short text message
1 MB      = 1,000 KB       â‰ˆ a photo (compressed), 1 minute of audio
1 GB      = 1,000 MB       â‰ˆ a movie (compressed)
1 TB      = 1,000 GB       â‰ˆ 1,000 movies
1 PB      = 1,000 TB       â‰ˆ what large companies store in a day

SPEED OF OPERATIONS (know these cold):
â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”
L1 cache (on CPU chip)          0.5 ns    "instant"
L2 cache (near CPU)               7 ns    "instant"
RAM read                        100 ns    still very fast
SSD read                        150 Âµs    1,500x slower than RAM
Network, same datacenter        0.5 ms    slow-ish
HDD seek                         10 ms    very slow
Network, cross-continent        150 ms    painfully slow
Network, cross-ocean            300 ms    very painful

THE KEY INSIGHT:
â”â”â”â”â”â”â”â”â”â”â”â”â”â”â”
RAM is ~1,500 times faster than SSD
SSD is ~70 times faster than HDD
Same datacenter network ~ SSD
Cross-continent network = nightmare

â†’ Cache = serve from RAM instead of SSD/DB
â†’ CDN = serve from nearby instead of cross-continent
```

| What | Approximate |
|---|---|
| Read 1MB from RAM | 0.25 ms |
| Read 1MB from SSD | 1 ms |
| Read 1MB from HDD | 20 ms |
| Redis GET operation | < 1 ms |
| DB query (cached plan, indexed) | 1-5 ms |
| DB query (cold, full scan) | 100-5000 ms |
| HTTP req (same city CDN) | 10-30 ms |
| HTTP req (cross-continent) | 150-300 ms |

---

## 19. Glossary

A plain-English definition of every term used in system design:

| Term | What it actually means |
|---|---|
| **Latency** | How long one request takes from start to finish. "High latency" = slow. |
| **Throughput** | How many requests your system handles per second. "High throughput" = handles lots. |
| **Scalability** | Can your system handle 10x more load by adding resources? |
| **Availability** | Is your system up and responding? Measured as a % of time. |
| **Reliability** | Does your system do what it's supposed to do, consistently? |
| **Durability** | If you save data, does it stay saved even after crashes? |
| **Consistency** | Do all nodes/users see the same data at the same time? |
| **Eventual consistency** | Data will be consistent across all nodes... but maybe not immediately. |
| **Redundancy** | Having backup components so one failure doesn't kill everything. |
| **Failover** | Automatic switch from a failed component to its backup. |
| **SPOF** | Single Point of Failure â€” one thing whose death kills the whole system. |
| **Stateless** | Server has no memory of previous requests. Each request contains everything needed. |
| **Idempotent** | Doing the same operation multiple times gives same result. Safe to retry. |
| **Horizontal scaling** | Add more machines (scale out). |
| **Vertical scaling** | Make existing machine bigger (scale up). |
| **Sharding** | Split data across multiple DB machines. Each holds a different slice. |
| **Replication** | Copy the SAME data to multiple machines. Different shards. |
| **Cache hit** | Requested data found in cache. Fast. |
| **Cache miss** | Requested data NOT in cache. Had to go to DB. Slow. |
| **Cache invalidation** | Deciding when to delete or update cached data because the source changed. |
| **TTL** | Time To Live â€” how long something is valid before expiring. |
| **Fan-out** | One event causing writes to many places (e.g. one tweet â†’ many timelines). |
| **Hot spot** | One shard/server getting way more traffic than others. |
| **Rate limiting** | Capping how many requests a client can make in a time window. |
| **Backpressure** | Signal from an overwhelmed component to its upstream to slow down. |
| **SLA** | Service Level Agreement â€” the uptime % you promise customers. |
| **SLO** | Service Level Objective â€” internal target for uptime/performance. |
| **P99 latency** | 99th percentile latency â€” "99% of requests complete within this time". |
| **CDN** | Content Delivery Network â€” global servers that cache your static content closer to users. |
| **Edge server** | A server physically close to the end user. CDN nodes are edge servers. |
| **Origin server** | Your main server. CDNs fetch from origin when they don't have a cached copy. |
| **Partition** | A network split between nodes that can't communicate with each other. |
| **Consensus** | Multiple distributed nodes agreeing on one value (very hard problem). |
| **Leader election** | Choosing one node to be the "primary" / "leader" in a cluster. |
| **Circuit breaker** | Stops calling a failing service and fails fast instead of waiting. Prevents cascade failures. |
| **Saga pattern** | A way to handle distributed transactions across microservices without a global lock. |
| **CQRS** | Command Query Responsibility Segregation â€” separate reads and writes into different models. |
| **Event sourcing** | Store all changes as events rather than just the current state. |
| **Bloom filter** | Probabilistic data structure: "Is this item definitely NOT in the set?" Used for cache pre-checks. |
| **Consistent hashing** | A way to distribute load across servers so adding/removing servers only affects a small % of data. |

---

## 20. Single Server Setup â€” where every system starts

Before load balancers, databases clusters, and CDNs â€” every system starts with one machine doing everything.

### What a single server looks like

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                    Your One Server                       â”‚
â”‚                                                          â”‚
â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”   â”‚
â”‚  â”‚  Web Server (Nginx / Apache)                     â”‚   â”‚
â”‚  â”‚  Listens on port 80/443, handles HTTP requests   â”‚   â”‚
â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜   â”‚
â”‚                            â†•                             â”‚
â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”   â”‚
â”‚  â”‚  Application Code (Node.js / Python / Java)      â”‚   â”‚
â”‚  â”‚  Your business logic â€” figures out what to do    â”‚   â”‚
â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜   â”‚
â”‚                            â†•                             â”‚
â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”   â”‚
â”‚  â”‚  Database (PostgreSQL / MySQL / MongoDB)          â”‚   â”‚
â”‚  â”‚  Stores all your data on the same disk           â”‚   â”‚
â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜   â”‚
â”‚                                                          â”‚
â”‚     Everything lives on ONE machine                      â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
                       â†‘
              All user traffic
```

### A request hitting a single server

```mermaid
sequenceDiagram
    participant U as ðŸ‘¤ User (Browser)
    participant DNS as ðŸŒ DNS
    participant S as ðŸ–¥ï¸ Single Server
    participant DB as ðŸ—„ï¸ Database (on same machine)

    U->>DNS: "What IP is myapp.com?"
    DNS-->>U: "It's 123.45.67.89"
    U->>S: GET /homepage
    S->>DB: SELECT articles FROM posts LIMIT 10
    DB-->>S: Here's the data
    S-->>U: Here's the HTML page
```

### Why it's great when you're starting out

```
âœ… Dead simple â€” one machine, one bill, one place to look
âœ… Fast to set up â€” no architecture decisions needed
âœ… Easy to debug â€” everything is right there
âœ… Cheap â€” a basic cloud VM costs Â£5â€“20/month
âœ… No network hops between components â†’ very fast response times
```

### When it starts to crack

```
10 users       â†’ Runs perfectly fine
1,000 users    â†’ A bit slow during busy periods
10,000 users   â†’ CPU maxed out, pages timing out
100,000 users  â†’ Server crashes regularly under load

But even before performance, there are two structural problems:

Problem 1: SINGLE POINT OF FAILURE
  If this one machine goes down (power cut, hardware failure,
  bad software deploy) â†’ your entire app is dead.
  No backup. No fallback. Just down.

Problem 2: DATA IS AT RISK
  Your database is on the same machine as your app.
  If the disk fails â†’ you lose everything.
  No replication. No backups (unless you set them up separately).

Problem 3: CAN'T SCALE SMARTLY
  If only your database is slow, you can't scale just that.
  You have to scale the whole machine â€” it's all one unit.
```

> **This is why the rest of system design exists** â€” solving these three problems as your app grows.

---

### The evolution path: single server â†’ production system

```
Stage 1: Single server          Stage 2: Separate the DB
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”              â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”   â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ App + DB       â”‚    â”€â”€â”€â”€â”€â”€â–º   â”‚ App Server   â”‚   â”‚ DB Server â”‚
â”‚ (one machine)  â”‚              â”‚              â”‚â”€â”€â–ºâ”‚           â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜              â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜   â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜

Stage 3: Add a cache            Stage 4: Scale horizontally
â”Œâ”€â”€â”€â”€â”€â”€â”€â”  â”Œâ”€â”€â”€â”€â”€â”€â”€â”            â”Œâ”€â”€â”€â”€â”€â”€â”         â”Œâ”€â”€â”€â”€â”€â”€â” â”Œâ”€â”€â”€â”€â”€â”€â”
â”‚ App   â”‚  â”‚ Cache â”‚            â”‚  LB  â”‚ â”€â”€â”€â”€â”€â”€â–º â”‚ App  â”‚ â”‚ App  â”‚
â”‚       â”‚â”€â–ºâ”‚(Redis)â”‚â”€â”€â–º DB      â”‚      â”‚          â”‚  1   â”‚ â”‚  2   â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”˜  â””â”€â”€â”€â”€â”€â”€â”€â”˜            â””â”€â”€â”€â”€â”€â”€â”˜          â””â”€â”€â”¬â”€â”€â”€â”˜ â””â”€â”€â”¬â”€â”€â”€â”˜
                                                      â””â”€â”€â–º DB â—„â”˜
```

---

## 21. API Design â€” the contract between systems

### What even is an API?

**API** stands for Application Programming Interface. Ignore the jargon. The concept is simple.

Think of a restaurant. You don't go into the kitchen and cook your own food. You look at the **menu**, order something, and a waiter brings it. The menu is the API â€” it's a defined list of things you can request, in a specific format, and you'll get a specific thing back.

```
The Restaurant Analogy:

You                  Waiter (API)              Kitchen (Server)
  â”‚                       â”‚                          â”‚
  â”œâ”€ "Can I get a burger?" â–º                          â”‚
  â”‚                       â”œâ”€â”€â”€ tells kitchen â”€â”€â”€â”€â”€â”€â”€â”€â–ºâ”‚
  â”‚                       â”‚                           â”‚
  â”‚                       â”‚â—„â”€â”€â”€ burger is ready â”€â”€â”€â”€â”€â”€â”¤
  â”‚â—„â”€â”€ Here's your burger â”¤                           â”‚
```

In software:

- **You** = a mobile app, website, or another service
- **The menu** = API documentation (what you can ask for)
- **The waiter** = the API (the defined interface)
- **The kitchen** = the server doing the actual work

---

### HTTP methods â€” the verbs of APIs

```
GET     â†’ "Give me something (read only, no side effects)"
          Example: GET /users/42  â†’  return user 42's profile

POST    â†’ "Create something new"
          Example: POST /orders  â†’  place a new order

PUT     â†’ "Replace this entirely"
          Example: PUT /users/42  â†’  replace the entire user record

PATCH   â†’ "Update just part of this"
          Example: PATCH /users/42  â†’  update just the email field

DELETE  â†’ "Remove this"
          Example: DELETE /users/42  â†’  delete user 42
```

---

### HTTP status codes â€” what the server says back

```
2xx â€” Success
  200 OK              â†’ Standard success
  201 Created         â†’ Resource was successfully created
  204 No Content      â†’ Success, but nothing to return (e.g. after DELETE)

4xx â€” Your fault (client error)
  400 Bad Request     â†’ You sent invalid data
  401 Unauthorized    â†’ You're not logged in (no valid token)
  403 Forbidden       â†’ You're logged in, but can't access this
  404 Not Found       â†’ That resource doesn't exist
  429 Too Many Requests â†’ You're being rate limited. Slow down.

5xx â€” Our fault (server error)
  500 Internal Server Error â†’ Something broke on the server
  503 Service Unavailable   â†’ Server is down or overloaded
```

---

### What a good API request and response look like

```
Creating a new order:
â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
REQUEST:
POST /api/v1/orders
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
Content-Type: application/json

{
  "product_id": 123,
  "quantity": 2,
  "delivery_address": "10 Downing St, London"
}

RESPONSE (201 Created):
{
  "order_id": "ord_abc123",
  "status": "confirmed",
  "estimated_delivery": "2024-12-25",
  "total_amount": 29.98,
  "created_at": "2024-12-01T09:00:00Z"
}
```

---

### Good API design principles

```
âœ… Use nouns for resources, not verbs:
   GET /orders          (good)
   GET /getOrders        (bad â€” "get" is in the URL, it's already a GET request)

âœ… Use nested URLs for relationships:
   GET /users/42/orders      (all orders for user 42)
   GET /orders/7/items       (all items in order 7)

âœ… Always version your API:
   /api/v1/users
   /api/v2/users
   (so you can change v2 without breaking apps still using v1)

âœ… Use consistent JSON structure:
   Success: { "data": {...}, "meta": {...} }
   Error:   { "error": "reason here", "code": 400 }

âœ… Support pagination â€” never return a million records at once:
   GET /posts?page=2&limit=20
   Response includes: { "data": [...], "next_cursor": "abc", "has_more": true }

âœ… Idempotency â€” safe operations should be safe to retry:
   GET /users/42  â†’  same result every time âœ…
   DELETE /orders/99  â†’  first call deletes it, second call: 404, but no duplicate harm âœ…
   POST /payments  â†’  must use an idempotency-key header to prevent double-charges âš ï¸
```

---

### API versioning strategies

```
URL versioning (most common):
  /api/v1/users
  /api/v2/users
  Simple, visible, easy to route

Header versioning:
  GET /api/users
  Accept: application/vnd.myapp.v2+json
  Clean URLs, but less obvious

Query param versioning:
  GET /api/users?version=2
  Easy to test in browser, but messy
```

> Version your API from day one. Changing a public API without versioning breaks every client that uses it.

---

## 22. API Protocols â€” the languages systems use to talk

An API protocol is the agreed-upon set of rules for how two systems exchange messages. Think of it like choosing a spoken language â€” both sides need to use the same one.

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                                                              â”‚
â”‚   REST        â†’ "Give me this resource"                      â”‚
â”‚   GraphQL     â†’ "Give me EXACTLY these fields"              â”‚
â”‚   gRPC        â†’ "Call this function on that machine"        â”‚
â”‚   WebSocket   â†’ "Keep the connection open, let's chat"      â”‚
â”‚   Webhook     â†’ "Call ME when something happens"            â”‚
â”‚                                                              â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

---

### REST â€” the most common

Standard request/response over HTTP. You ask for a resource, the server responds.

- Simple, universally understood
- Stateless â€” each request stands alone
- Uses HTTP verbs (GET, POST, PUT, DELETE)
- Data in JSON format
- Great for: public APIs, mobile apps, web frontends

---

### GraphQL â€” ask for exactly what you want

The client specifies exactly what data it needs. Server returns only that.

- Solves "over-fetching" (getting too much data) and "under-fetching" (not enough in one call)
- One endpoint: `POST /graphql`
- Great for: complex UIs, mobile apps where bandwidth matters

---

### gRPC â€” speed for internal services

Lets one service call a function on another service as if it were local code. Uses Protocol Buffers (binary format) instead of JSON â€” much faster.

- Very fast (binary, not text)
- Strongly typed â€” both sides know exactly what shape the data is
- Great for: internal microservice communication where speed matters

---

### WebSockets â€” real-time two-way connection

A persistent, live connection between client and server. Either side can send messages at any time.

- Not request/response â€” true real-time
- Great for: chat apps, live notifications, multiplayer games, stock tickers

```mermaid
graph TB
    subgraph REST - Request / Response
        C1["Client"] -->|"GET /posts (request)"| S1["Server"]
        S1 -->|"[list of posts] (response)"| C1
    end

    subgraph WebSocket - Persistent open channel
        C2["Client"] <-->|"Messages flow BOTH ways\nat any time, in real-time"| S2["Server"]
    end

    subgraph gRPC - Remote function call
        C3["Service A"] -->|"getUserById(42)"| S3["Service B"]
        S3 -->|"User{id:42, name:'Alice'}"| C3
    end
```

---

### Webhooks â€” the server calls you

Instead of you repeatedly asking "is it done yet?", the server calls you when something happens.

```
Polling (wasteful):                  Webhook (efficient):

Every 5 seconds, you ask:            Server calls YOU when it's ready:
  "Is my payment done?"              POST https://yourapp.com/webhook
  "Is my payment done?"              { "event": "payment.completed",
  "Is my payment done?"                "order_id": "ord_abc123" }
  "Is my payment done?"
  "Yes, just now!" â† only useful
                    reply in ~20

Like calling someone every 5 min     Like asking them to text you
to ask "are you available?"          when they're free
```

---

### Protocol comparison

| Protocol      | Style                    | Format            | Best for                             |
| ------------- | ------------------------ | ----------------- | ------------------------------------ |
| **REST**      | Request / Response       | JSON              | Public APIs, CRUD apps               |
| **GraphQL**   | Request / Response       | JSON              | Complex queries, mobile bandwidth    |
| **gRPC**      | Request / Response       | Binary (Protobuf) | High-speed internal microservices    |
| **WebSocket** | Persistent bidirectional | Text or Binary    | Chat, live feeds, multiplayer games  |
| **Webhook**   | Server pushes to client  | JSON              | Event notifications (Stripe, GitHub) |

---

```
â•”â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•—
â•‘                                                               â•‘
â•‘   The best system is the simplest one that solves            â•‘
â•‘   your problem today. Not the one designed for a             â•‘
â•‘   scale you may never reach.                                 â•‘
â•‘                                                              â•‘
â•‘   Start simple. Scale when the pain is real.                 â•‘
â•‘   Complexity is a cost, not a feature.                       â•‘
â•‘                                                               â•‘
â•šâ•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•
```

---

_Part of the AI_Projects learning series._
2. [The Building Blocks](#2-the-building-blocks)
3. [Scalability â€” how do you grow?](#3-scalability--how-do-you-grow)
4. [Availability â€” how do you stay up?](#4-availability--how-do-you-stay-up)
5. [CAP Theorem â€” the impossible triangle](#5-cap-theorem--the-impossible-triangle)
6. [Databases â€” choosing the right storage](#6-databases--choosing-the-right-storage)
7. [Caching â€” stop asking the same question twice](#7-caching--stop-asking-the-same-question-twice)
8. [Load Balancers â€” the traffic cops](#8-load-balancers--the-traffic-cops)
9. [CDN â€” the world is big, servers are slow](#9-cdn--the-world-is-big-servers-are-slow)
10. [Message Queues â€” don't do things right now](#10-message-queues--dont-do-things-right-now)
11. [API Gateway â€” the front door](#11-api-gateway--the-front-door)
12. [Rate Limiting â€” slowing down the greedy](#12-rate-limiting--slowing-down-the-greedy)
13. [Database Sharding â€” cutting the data pie](#13-database-sharding--cutting-the-data-pie)
14. [Replication â€” keeping backups alive](#14-replication--keeping-backups-alive)
15. [Microservices vs Monolith](#15-microservices-vs-monolith)
16. [Real World Examples](#16-real-world-examples)
17. [The System Design Interview Playbook](#17-the-system-design-interview-playbook)

---

## 1. What even is System Design?

Imagine you're building a food delivery app. Day 1: just you and your friends use it. One server, one database, everything is fine.

Now imagine 10 million people use it on New Year's Eve at the same time. What happens?

- Your one server catches fire (figuratively)
- Your database can't answer 10 million questions per second
- One bug takes your entire app down
- Someone in Australia is waiting 8 seconds for a page to load because your server is in New York

**System design is the art of deciding in advance how your system will handle all of this â€” before it happens.**

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                         YOUR APP                                â”‚
â”‚                                                                  â”‚
â”‚   Small scale              vs           Large scale             â”‚
â”‚                                                                  â”‚
â”‚   1 user  â†’  1 server         1M users â†’ ???                    â”‚
â”‚   1 DB    â†’  simple           multiple DBs, caches, queues      â”‚
â”‚   1 team  â†’  knows it all     100 teams â†’ microservices         â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

---

## 2. The Building Blocks

Every large system is built from the same set of Lego bricks. Know these, and you can design almost anything.

```mermaid
graph TD
    User["ðŸ‘¤ User / Browser / App"]
    DNS["ðŸŒ DNS\n(Finds the address)"]
    CDN["ðŸ“¦ CDN\n(Nearby static files)"]
    LB["âš–ï¸ Load Balancer\n(Splits traffic)"]
    API["ðŸšª API Gateway\n(Front door)"]
    S1["ðŸ–¥ï¸ Server 1"]
    S2["ðŸ–¥ï¸ Server 2"]
    S3["ðŸ–¥ï¸ Server 3"]
    Cache["âš¡ Cache\n(Fast answers)"]
    DB["ðŸ—„ï¸ Primary Database"]
    DBR["ðŸ—„ï¸ Replica DB\n(Read only)"]
    Queue["ðŸ“¬ Message Queue\n(Jobs to do later)"]
    Worker["âš™ï¸ Worker / Background job"]

    User -->|"types URL"| DNS
    User -->|"static files"| CDN
    DNS -->|"IP address"| LB
    LB --> API
    API --> S1
    API --> S2
    API --> S3
    S1 --> Cache
    S2 --> Cache
    S3 --> Cache
    Cache -->|"cache miss"| DB
    DB --> DBR
    S1 --> Queue
    Queue --> Worker
    Worker --> DB
```

| Block                       | What it does                                                                                | Real-world example                                                |
| --------------------------- | ------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| **DNS**                     | Converts `google.com` into an IP address your computer can actually connect to              | Like a phone book â€” name â†’ number                                 |
| **Load Balancer**           | Spreads requests across multiple servers so no single one gets overwhelmed                  | A restaurant manager seating customers across all tables          |
| **CDN**                     | Keeps copies of your static files (images, videos, CSS) in servers close to users worldwide | Netflix serving videos from a server in your city, not California |
| **Cache**                   | Stores answers to common questions in fast memory so you don't hit the database every time  | Post-its on your desk vs going to the filing room                 |
| **Database**                | Persistent storage â€” data lives here even when the server restarts                          | The actual filing room                                            |
| **Message Queue**           | A to-do list for your system â€” tasks get added and processed in order, at their own pace    | A ticket queue at a help desk                                     |
| **API Gateway**             | Single entry point for all incoming requests â€” handles auth, routing, rate limiting         | Reception at an office building                                   |
| **Worker / Background job** | Picks up tasks from the queue and does them slowly (sending emails, processing images)      | The people who actually handle each ticket                        |

---

## 3. Scalability â€” how do you grow?

Scalability is your system's ability to handle more load without falling over.

There are exactly two ways to scale:

### Vertical Scaling â€” make the machine bigger

Buy a beefier server. More CPU, more RAM, more disk.

```
Before:                After:
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”          â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ Server   â”‚  â”€â”€â”€â”€â”€â”€â–º â”‚ BIGGER Server    â”‚
â”‚ 4 cores  â”‚          â”‚ 64 cores         â”‚
â”‚ 8GB RAM  â”‚          â”‚ 512GB RAM        â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜          â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

âœ… Simple â€” no code changes needed  
âœ… Works well up to a point  
âŒ Has a hard ceiling â€” you can only make a machine so big  
âŒ Single point of failure â€” if this one machine dies, everything dies  
âŒ Expensive â€” high-end hardware costs exponentially more

---

### Horizontal Scaling â€” add more machines

Instead of making one server bigger, add more servers and split the load.

```
Before:                After:
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”          â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ Server   â”‚  â”€â”€â”€â”€â”€â”€â–º â”‚ Server 1 â”‚  â”‚ Server 2 â”‚  â”‚ Server 3 â”‚
â”‚ 4 cores  â”‚          â”‚ 4 cores  â”‚  â”‚ 4 cores  â”‚  â”‚ 4 cores  â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜          â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
                              â–²            â–²            â–²
                              â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
                                      Load Balancer
```

âœ… No ceiling â€” just keep adding more  
âœ… No single point of failure â€” one server dies, others keep going  
âœ… Cheaper at scale â€” commodity hardware  
âŒ More complex â€” your app needs to be stateless (no session stored on one machine)  
âŒ Needs a load balancer  
âŒ Distributed systems are hard

> **Rule of thumb:** Start vertical, switch to horizontal when vertical gets painful or expensive.

---

## 4. Availability â€” how do you stay up?

**Availability** is measured as a percentage of time your system is up and running.

| Availability | Downtime per year | Called                   |
| ------------ | ----------------- | ------------------------ |
| 90%          | 36.5 days         | Terrible                 |
| 99%          | 3.65 days         | Okay                     |
| 99.9%        | 8.7 hours         | Good ("three nines")     |
| 99.99%       | 52 minutes        | Very good ("four nines") |
| 99.999%      | 5 minutes         | Excellent ("five nines") |

Big companies like AWS, Google aim for 99.99% or better. Getting from 99.9% to 99.999% is insanely hard and expensive.

### How do you get high availability?

**Redundancy** â€” don't have a single component that, if it fails, takes everything down.

```mermaid
graph LR
    subgraph Bad - Single Point of Failure
        U1[User] --> S_bad[Server]
        S_bad --> DB_bad[(Database)]
    end

    subgraph Good - Redundant
        U2[User] --> LB2[Load Balancer]
        LB2 --> S2a[Server A]
        LB2 --> S2b[Server B]
        S2a --> DB_primary[(Primary DB)]
        S2b --> DB_primary
        DB_primary -- "syncs to" --> DB_replica[(Replica DB)]
    end
```

**Failover** â€” when something dies, automatically switch to the backup.  
**Health checks** â€” constantly ping each component; remove it from rotation if it stops responding.

---

## 5. CAP Theorem â€” the impossible triangle

This is one of the most famous ideas in distributed systems. It says:

> **In a distributed system, you can only ever guarantee 2 out of these 3 things at the same time.**

```
                    â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
                    â”‚  Consistency    â”‚
                    â”‚                 â”‚
                    â”‚ "Every read     â”‚
                    â”‚  gets the       â”‚
                    â”‚  latest write"  â”‚
                    â””â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”˜
                             â”‚
                    Pick any â”‚ two
                            /â”‚\
                           / â”‚ \
                          /  â”‚  \
          â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”   â”‚   â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
          â”‚ Availability â”‚   â”‚   â”‚  Partition   â”‚
          â”‚              â”‚   â”‚   â”‚  Tolerance   â”‚
          â”‚ "System staysâ”‚   â”‚   â”‚              â”‚
          â”‚  up even if  â”‚   â”‚   â”‚ "Works even  â”‚
          â”‚  some nodes  â”‚   â”‚   â”‚  if the      â”‚
          â”‚  fail"       â”‚   â”‚   â”‚  network     â”‚
          â”‚              â”‚   â”‚   â”‚  splits"     â”‚
          â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜   â”‚   â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

**In practice:** Network failures (partitions) happen â€” you can't avoid them. So every real system must be Partition Tolerant. That means you're actually choosing between **CP** (consistent but sometimes unavailable) or **AP** (always available but possibly stale).

| Choice | What you get                                                   | Examples                              |
| ------ | -------------------------------------------------------------- | ------------------------------------- |
| **CP** | Correct data, but might reject requests during a network split | HBase, MongoDB (by config), Zookeeper |
| **AP** | Always responds, but might give you slightly old data          | Cassandra, DynamoDB, CouchDB          |

> The classic example: your bank balance should be CP (you'd rather get an error than see the wrong balance). Your Twitter feed can be AP (seeing a tweet 2 seconds late is fine).

---

## 6. Databases â€” choosing the right storage

### SQL (Relational) â€” structured, strict, powerful

Think of it like a spreadsheet â€” rows, columns, strict rules. Data lives in tables and rows relate to each other.

```
Users Table:              Orders Table:
â”Œâ”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”         â”Œâ”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ ID â”‚ Name     â”‚         â”‚ ID â”‚ User_ID â”‚ Amount   â”‚
â”œâ”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤         â”œâ”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚  1 â”‚ Alice    â”‚â—„â”€â”€â”€â”€â”€â”€â”€â”€â”‚  1 â”‚    1    â”‚  $49.99  â”‚
â”‚  2 â”‚ Bob      â”‚â—„â”€â”€â”€â”    â”‚  2 â”‚    2    â”‚  $12.50  â”‚
â””â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜    â””â”€â”€â”€â”€â”‚  3 â”‚    2    â”‚  $99.00  â”‚
                          â””â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

âœ… ACID guarantees (Atomicity, Consistency, Isolation, Durability)  
âœ… Powerful JOIN queries  
âœ… Great for complex relationships  
âŒ Hard to scale horizontally  
âŒ Schema changes are painful

**Use SQL when:** You have structured data with clear relationships â€” banking, e-commerce, user accounts.

---

### NoSQL â€” flexible, fast, scalable

No fixed schema. Different flavours for different needs.

```mermaid
graph TD
    NoSQL["NoSQL Databases"]

    NoSQL --> DocDB["ðŸ“„ Document\n(MongoDB, Firestore)\nStores JSON-like docs\nGood for: user profiles, catalogs"]
    NoSQL --> KV["ðŸ”‘ Key-Value\n(Redis, DynamoDB)\nLike a dictionary\nGood for: caching, sessions"]
    NoSQL --> Wide["ðŸ“Š Wide Column\n(Cassandra, HBase)\nFlexible columns per row\nGood for: time-series, IoT data"]
    NoSQL --> Graph["ðŸ•¸ï¸ Graph\n(Neo4j)\nNodes and edges\nGood for: social networks, fraud detection"]
```

| Type            | Think of it as                                          | Best for                              |
| --------------- | ------------------------------------------------------- | ------------------------------------- |
| **Document**    | A filing cabinet full of folders, each folder different | User profiles, product catalogs       |
| **Key-Value**   | A massive dictionary / hash map                         | Sessions, caches, leaderboards        |
| **Wide Column** | A spreadsheet where each row can have different columns | Analytics, logs, time-series          |
| **Graph**       | A mind map â€” nodes connected by relationships           | Social graphs, recommendation engines |

### ACID vs BASE

| Property | ACID (SQL)                                | BASE (NoSQL)                                        |
| -------- | ----------------------------------------- | --------------------------------------------------- |
| **A**    | Atomic â€” all or nothing                   | Basically Available â€” always responds               |
| **C/S**  | Consistent â€” rules always apply           | Soft state â€” may not be consistent right now        |
| **I/E**  | Isolated â€” transactions don't interfere   | Eventually consistent â€” will be right... eventually |
| **D**    | Durable â€” committed data survives crashes | â€”                                                   |

> Banking uses ACID. Your Instagram likes counter uses BASE.

---

## 7. Caching â€” stop asking the same question twice

A cache is a layer of fast storage that keeps a copy of data you've recently fetched, so you don't have to go all the way to the database again.

```mermaid
sequenceDiagram
    participant U as User
    participant S as Server
    participant C as Cache (Redis)
    participant DB as Database

    Note over C,DB: First request â€” cache is empty
    U->>S: "Get profile for user 42"
    S->>C: Check cache
    C-->>S: âŒ Not found
    S->>DB: SELECT * FROM users WHERE id=42
    DB-->>S: {name: "Alice", ...}
    S->>C: Store in cache (expires in 10 min)
    S-->>U: Here's Alice's profile

    Note over C,DB: Second request â€” cache has it
    U->>S: "Get profile for user 42"
    S->>C: Check cache
    C-->>S: âœ… Found! {name: "Alice", ...}
    S-->>U: Here's Alice's profile (10x faster)
```

### Cache Eviction Policies â€” what to delete when the cache is full?

| Policy                          | How it works                             | Analogy                                          |
| ------------------------------- | ---------------------------------------- | ------------------------------------------------ |
| **LRU** (Least Recently Used)   | Remove whatever was accessed longest ago | Throw out the book you haven't touched in months |
| **LFU** (Least Frequently Used) | Remove whatever is accessed least often  | Throw out the book you've only read once         |
| **FIFO**                        | Remove oldest entry regardless of use    | Queue â€” first in, first out                      |
| **TTL**                         | Each entry expires after a set time      | Milk with an expiry date                         |

### Cache Invalidation â€” the hardest problem in CS

> _"There are only two hard things in Computer Science: cache invalidation and naming things."_ â€” Phil Karlton

When the data in your database changes, how does the cache know to update? The main strategies:

```
Write-Through:          Write-Around:           Write-Back:
Write to cache AND      Write to DB only.       Write to cache only.
DB at the same time.    Cache fills on read.    DB updated later.

Consistent âœ…           Cache is cold âš ï¸        Fast writes âœ…
Slow writes âš ï¸          DB not overwhelmed âœ…   Risk of data loss âš ï¸
```

### Where to put your cache?

```
Client-side â†’ Browser caches HTML, images, API responses (localStorage)
         â†“
CDN cache â†’ Cloudflare caches your static assets globally
         â†“
Server-side â†’ Redis/Memcached sits between your server and DB
         â†“
Database â†’ Query result cache inside the DB itself
```

---

## 8. Load Balancers â€” the traffic cops

A load balancer sits in front of your servers and decides which server handles each incoming request.

```
Without Load Balancer:          With Load Balancer:

All traffic â†’ Server 1          Traffic â†’ Load Balancer
              (overwhelmed)                    â”‚
              Server 2 (idle)                  â”œâ”€â”€â–º Server 1 (33%)
              Server 3 (idle)                  â”œâ”€â”€â–º Server 2 (33%)
                                               â””â”€â”€â–º Server 3 (33%)
```

### How does it decide where to send traffic?

| Algorithm                | How it works                                                     | Best for                         |
| ------------------------ | ---------------------------------------------------------------- | -------------------------------- |
| **Round Robin**          | Server 1, then 2, then 3, then back to 1...                      | Servers with similar capacity    |
| **Weighted Round Robin** | Server 1 gets 50%, Server 2 gets 30%, Server 3 gets 20%          | Servers with different power     |
| **Least Connections**    | Send to whichever server has fewest active connections right now | Long-running requests            |
| **IP Hash**              | User's IP always goes to the same server                         | When you need session stickiness |
| **Random**               | Pick any server at random                                        | Simple, works surprisingly well  |

### Layer 4 vs Layer 7

```
Layer 4 (Transport):            Layer 7 (Application):
Routes based on IP and port     Routes based on URL, headers, cookies
Faster (just TCP/UDP)           Smarter (knows about HTTP)
Can't look inside the request   Can route /api to one server,
                                /images to another
```

---

## 9. CDN â€” the world is big, servers are slow

A **Content Delivery Network** is a global network of servers that keep copies of your static files (images, videos, CSS, JavaScript) close to your users.

```mermaid
graph TB
    Origin["ðŸ¢ Origin Server\n(New York)"]

    EU["ðŸ‡ªðŸ‡º CDN â€” Europe\n(Amsterdam)"]
    AS["ðŸŒ CDN â€” Asia\n(Singapore)"]
    AU["ðŸ‡¦ðŸ‡º CDN â€” Australia\n(Sydney)"]

    UserEU["ðŸ‘¤ User in Paris"]
    UserAS["ðŸ‘¤ User in Tokyo"]
    UserAU["ðŸ‘¤ User in Sydney"]

    Origin -->|"first request, file stored"| EU
    Origin -->|"first request, file stored"| AS
    Origin -->|"first request, file stored"| AU

    UserEU -->|"image loads in 20ms"| EU
    UserAS -->|"image loads in 15ms"| AS
    UserAU -->|"image loads in 10ms"| AU
```

Without CDN: User in Tokyo fetches image from New York â†’ 200ms+ latency  
With CDN: User in Tokyo fetches image from Singapore â†’ 15ms latency

**CDN handles:**

- Images, videos, CSS, JavaScript files
- HTML pages that don't change often
- Large file downloads
- Live streaming (with streaming CDNs)

**Not good for:**

- Personalized / dynamic content
- Real-time data (stock prices, chat messages)

---

## 10. Message Queues â€” don't do things right now

Sometimes you don't need to do something immediately. You just need to make sure it gets done eventually.

**The problem without queues:**

```
User uploads photo â†’ Server must:
  1. Save photo      (0.1s)
  2. Resize photo    (2s)    â† User is waiting...
  3. Add watermark   (1s)    â† Still waiting...
  4. Update DB       (0.1s)  â† Now the UI responds
  Total: 3.2s of user waiting for things they don't even need to see
```

**The solution with queues:**

```
User uploads photo â†’ Server:
  1. Save original photo    (0.1s)
  2. Put "process photo" job on queue
  3. Respond to user: "Upload successful!" â† User is happy in 0.1s

Meanwhile, in the background:
  Worker picks up job â†’ Resize â†’ Watermark â†’ Done
```

```mermaid
graph LR
    P1["ðŸ“± App Server 1\n(Producer)"]
    P2["ðŸ“± App Server 2\n(Producer)"]
    Q["ðŸ“¬ Message Queue\n(RabbitMQ / Kafka / SQS)"]
    W1["âš™ï¸ Worker 1\n(Consumer)"]
    W2["âš™ï¸ Worker 2\n(Consumer)"]
    W3["âš™ï¸ Worker 3\n(Consumer)"]

    P1 -->|"add job"| Q
    P2 -->|"add job"| Q
    Q -->|"job assigned"| W1
    Q -->|"job assigned"| W2
    Q -->|"job assigned"| W3
```

### Why queues are great

âœ… **Decoupling** â€” the sender doesn't care who processes it, or when  
âœ… **Buffering** â€” spike of 50,000 messages? Queue holds them, workers process at their own pace  
âœ… **Retry** â€” if a worker fails, the message goes back in the queue  
âœ… **Async** â€” user gets a fast response, heavy work happens later

### Kafka vs RabbitMQ

|                    | Kafka                                                 | RabbitMQ                                           |
| ------------------ | ----------------------------------------------------- | -------------------------------------------------- |
| **Model**          | Log-based â€” messages are retained and can be replayed | Traditional queue â€” message deleted after consumed |
| **Scale**          | Millions of messages/sec                              | Hundreds of thousands/sec                          |
| **Use case**       | Event streaming, logs, audit trails                   | Task queues, async jobs                            |
| **Think of it as** | A receipt printer roll â€” keeps all history            | A to-do list â€” tick it off, it's gone              |

---

## 11. API Gateway â€” the front door

In a microservices world, you might have 50 different services. You don't want users calling each one directly. The **API Gateway** is the single front door.

```mermaid
graph LR
    Client["ðŸ“± Mobile App\n/ Web Browser"]

    GW["ðŸšª API Gateway"]

    Auth["ðŸ” Auth Service"]
    Users["ðŸ‘¤ User Service"]
    Orders["ðŸ“¦ Order Service"]
    Payments["ðŸ’³ Payment Service"]
    Notifications["ðŸ”” Notification Service"]

    Client -->|"all requests go here"| GW
    GW -->|"POST /login"| Auth
    GW -->|"GET /user/42"| Users
    GW -->|"POST /order"| Orders
    GW -->|"POST /pay"| Payments
    GW -->|"any service"| Notifications
```

The API Gateway does a lot of heavy lifting:

- **Authentication** â€” Is this user logged in? Valid token?
- **Rate limiting** â€” Has this user made too many requests?
- **Routing** â€” Which microservice handles this endpoint?
- **SSL termination** â€” HTTPS ends here; internal traffic is plain HTTP
- **Request logging** â€” Log everything in one place
- **Response caching** â€” Cache common responses

---

## 12. Rate Limiting â€” slowing down the greedy

Without rate limiting, a single user (or bot) could make millions of requests and bring your system down.

```
Without rate limiting:          With rate limiting:

Bot fires 100,000 req/s         Bot fires 100,000 req/s
         â†“                               â†“
   Server struggles             API Gateway: "You're allowed
   Legitimate users                100 req/min. You've hit it."
   get slow responses                      â†“
   or errors                      First 100 requests: go through
                                  Rest: âŒ 429 Too Many Requests
```

### Common algorithms

**Token Bucket** â€” The most common. You have a bucket that fills with tokens at a steady rate (e.g. 10 per second). Each request uses one token. When the bucket is empty, requests are rejected.

```
Bucket capacity: 100 tokens
Fill rate: 10 tokens/second

Normal user (5 req/sec):     ðŸª£ Bucket never empties â†’ all good
Burst user (50 req in 1s):   ðŸª£ Uses 50 tokens â†’ still has 50 left
Bot (1000 req/sec):          ðŸª£ Empty after 10 requests â†’ 429 error
```

**Fixed Window** â€” Count requests per fixed time window (e.g. 100 per minute). Simple but has a flaw: someone can send 100 at 11:59 and 100 at 12:00 â€” 200 in 2 seconds.

**Sliding Window** â€” Like fixed window but rolls continuously. Solves the boundary problem. More accurate, slightly more complex.

---

## 13. Database Sharding â€” cutting the data pie

What happens when your database gets too big for one machine? You have 10 billion rows â€” queries are slow, disk is full.

**Sharding** splits your data across multiple database machines (shards), each holding a piece.

```mermaid
graph TD
    App["Application Server"]
    Router["Shard Router\n(Which shard has user X?)"]

    DB1["ðŸ—„ï¸ Shard 1\nUsers Aâ€“H\n(2.5 billion rows)"]
    DB2["ðŸ—„ï¸ Shard 2\nUsers Iâ€“P\n(2.5 billion rows)"]
    DB3["ðŸ—„ï¸ Shard 3\nUsers Qâ€“Z\n(2.5 billion rows)"]

    App --> Router
    Router -->|"user starts with A-H"| DB1
    Router -->|"user starts with I-P"| DB2
    Router -->|"user starts with Q-Z"| DB3
```

### Shard Key strategies

| Strategy            | How                                           | Problem                                               |
| ------------------- | --------------------------------------------- | ----------------------------------------------------- |
| **Range-based**     | Users A-H â†’ Shard 1, I-P â†’ Shard 2            | Uneven â€” lots of "Smith" â†’ Shard 2 overloaded         |
| **Hash-based**      | hash(user_id) % 3 â†’ shard number              | Even distribution, but range queries are hard         |
| **Directory-based** | A lookup table maps each key to a shard       | Flexible, but the lookup table itself is a bottleneck |
| **Geography-based** | European users â†’ EU shard, Asian â†’ Asia shard | Great for latency + compliance (GDPR)                 |

### The dark side of sharding

âŒ **Cross-shard queries are painful** â€” "find all users who bought product X" now has to query 3 databases and merge results  
âŒ **No cross-shard transactions** â€” can't do ACID across shards easily  
âŒ **Resharding is hard** â€” if you add a 4th shard later, you have to move data around  
âŒ **Complexity** â€” your app now needs to know which shard to talk to

> Only shard when you have to. Most apps never need it.

---

## 14. Replication â€” keeping backups alive

Replication is keeping copies of your database on multiple machines. Different goal from sharding â€” sharding splits data, replication duplicates it.

```mermaid
graph LR
    W["âœï¸ Write request"] --> Primary["ðŸ—„ï¸ Primary DB\n(read + write)"]
    Primary -->|"syncs changes"| R1["ðŸ—„ï¸ Replica 1\n(read only)"]
    Primary -->|"syncs changes"| R2["ðŸ—„ï¸ Replica 2\n(read only)"]
    Primary -->|"syncs changes"| R3["ðŸ—„ï¸ Replica 3\n(read only)"]

    App["Application"] -->|"all reads"| R1
    App -->|"all reads"| R2
    App -->|"all reads"| R3
    App -->|"all writes"| Primary
```

### Sync vs Async replication

**Synchronous** â€” Primary waits for replica to confirm before returning "success." Data is always consistent. But slower, and if replica is down, primary is stuck.

**Asynchronous** â€” Primary returns success immediately, replica catches up when it can. Faster, but there's a window where replica might be slightly behind. If primary crashes, you might lose that last bit of data.

### What happens when Primary dies?

**Failover** â€” A replica gets promoted to become the new primary. Either:

- **Automatic failover** â€” System detects primary is gone and promotes a replica (seconds to minutes)
- **Manual failover** â€” A human intervenes (slower, but more controlled)

---

## 15. Microservices vs Monolith

### The Monolith â€” one big app

Everything is one codebase, one deployment, one database.

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚            Monolith App             â”‚
â”‚                                     â”‚
â”‚  User Module    Order Module        â”‚
â”‚  Auth Module    Payment Module      â”‚
â”‚  Search Module  Notification Module â”‚
â”‚                                     â”‚
â”‚           One Database              â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
         Deployed as one unit
```

âœ… Simple to develop when small  
âœ… Easy to test end-to-end  
âœ… No network calls between modules  
âœ… One deploy, one place to look  
âŒ As it grows, becomes a mess  
âŒ One bug can take down everything  
âŒ Can only scale the whole thing, even if only one part is slow  
âŒ Different teams stepping on each other's code

---

### Microservices â€” many small apps

Each service is its own independent codebase, database, and deployment.

```mermaid
graph TB
    GW["ðŸšª API Gateway"]

    US["ðŸ‘¤ User Service\n(Node.js + PostgreSQL)"]
    OS["ðŸ“¦ Order Service\n(Python + MongoDB)"]
    PS["ðŸ’³ Payment Service\n(Java + MySQL)"]
    NS["ðŸ”” Notification Service\n(Go + Redis)"]
    SS["ðŸ” Search Service\n(Elasticsearch)"]

    GW --> US
    GW --> OS
    GW --> PS
    GW --> NS
    GW --> SS

    OS -->|"payment event"| PS
    PS -->|"payment done event"| NS
```

âœ… Each service scales independently (Payment service is slow? Scale only that)  
âœ… One service dying doesn't take down others  
âœ… Teams work independently  
âœ… Can use different tech for each service  
âŒ Network calls between services (latency + failure risk)  
âŒ Much more complex to manage  
âŒ Distributed tracing / debugging is hard  
âŒ Data consistency across services is painful

> **Start with a monolith. Break it into microservices only when the pain is real.**

---

## 16. Real World Examples

### How Twitter / X works (simplified)

The hard problem: when a celebrity with 100 million followers tweets, 100 million timelines need updating instantly.

```mermaid
graph LR
    User["Celebrity\ntweets"]
    TW["Tweet Service\n(saves tweet)"]
    FO["Fanout Service\n(figures out who follows them)"]
    Q["Message Queue"]
    W["Timeline Workers\n(millions of writes)"]
    Cache["Timeline Cache\nfor each user"]
    R["Read service\n(assembles timeline)"]
    FU["Followers\nopen app"]

    User -->|"POST /tweet"| TW
    TW --> FO
    FO -->|"push to queue"| Q
    Q --> W
    W -->|"write to each follower's timeline"| Cache
    FU --> R
    R --> Cache
```

**The tricky part:** If you have 100M followers, writing to all their timelines takes time. Twitter actually has two strategies:

- **Push model (fanout on write)** â€” Pre-compute everyone's timeline when a tweet is posted. Fast reads, expensive writes.
- **Pull model (fanout on read)** â€” Just store the tweet. When someone opens the app, compute their timeline on the fly. Cheap writes, expensive reads.

Twitter uses a **hybrid** â€” push for normal users, pull for celebrities.

---

### How Netflix works (simplified)

The hard problem: serve video to 260+ million subscribers worldwide without buffering.

```
1. You hit Netflix.com â†’ CDN serves the static page instantly
2. You search "Stranger Things" â†’ Search service (Elasticsearch)
3. You hit Play â†’ API Gateway routes to Video Service
4. Video service checks: what's your internet speed? Device? Location?
5. Picks the right quality (4K / 1080p / 720p) and codec
6. Fetches video from the CDN server CLOSEST to you
7. Every 30 seconds your player reports back stats
8. If your bandwidth drops, quality automatically lowers (no buffering)
```

Netflix encodes every show in **dozens of different formats and qualities** in advance, stored on CDN servers in 200+ countries.

---

### How Uber works (simplified)

The hard problem: match a driver to a rider in real time, with both moving.

```mermaid
sequenceDiagram
    participant R as Rider
    participant API as API Gateway
    participant US as User Service
    participant MS as Matching Service
    participant DS as Driver Location DB
    participant D as Driver App

    R->>API: "I need a ride from A to B"
    API->>US: Authenticate rider
    API->>MS: Find nearby drivers
    MS->>DS: Query drivers within 2km of rider
    DS-->>MS: [Driver 1: 0.5km, Driver 2: 1.2km, ...]
    MS-->>API: Best match: Driver 1
    API->>D: "New ride request"
    D-->>API: "Accepted"
    API-->>R: "Driver on the way"

    loop Every 4 seconds
        D->>DS: Update my location
    end
```

The driver location system needs to handle millions of location updates every second. Uber uses a custom geospatial index to quickly find drivers near any location.

---

## 17. The System Design Interview Playbook

When you're asked to design a system in an interview (or in real life), follow this framework:

```
Step 1: CLARIFY (5 min)
â”œâ”€â”€ Who uses it? How many?
â”œâ”€â”€ What are the core features? (don't try to design everything)
â”œâ”€â”€ Read-heavy or write-heavy?
â”œâ”€â”€ Any special requirements? (consistency, latency, global?)
â””â”€â”€ Scale: requests/sec, data size, user count

Step 2: ESTIMATE (2 min)
â”œâ”€â”€ Daily active users â†’ requests per second
â”œâ”€â”€ Data per user â†’ total storage needed
â””â”€â”€ These rough numbers guide your design choices

Step 3: HIGH LEVEL DESIGN (10 min)
â”œâ”€â”€ Draw the basic flow: Client â†’ Load Balancer â†’ Servers â†’ DB
â”œâ”€â”€ Identify the core components needed
â””â”€â”€ Walk through a typical user request end to end

Step 4: DEEP DIVE (15 min)
â”œâ”€â”€ Pick the hardest part and go deep
â”œâ”€â”€ Database schema â€” what tables/documents?
â”œâ”€â”€ APIs â€” what endpoints?
â”œâ”€â”€ Where do bottlenecks happen?
â””â”€â”€ How do you fix them? (cache? shard? queue?)

Step 5: HANDLE EDGE CASES (5 min)
â”œâ”€â”€ What if the database goes down?
â”œâ”€â”€ What if traffic spikes 10x suddenly?
â”œâ”€â”€ What if a region loses connectivity?
â””â”€â”€ Security â€” how do you prevent abuse?
```

### Quick cheat sheet

| Problem                                    | Solution                                  |
| ------------------------------------------ | ----------------------------------------- |
| Single server overloaded                   | Horizontal scaling + Load Balancer        |
| Database too slow                          | Cache (Redis) in front of it              |
| Database too big                           | Sharding                                  |
| Single point of failure                    | Replication + Failover                    |
| Users far from server                      | CDN for static, edge servers for dynamic  |
| Slow user experience from heavy operations | Message Queue + async processing          |
| Too many services to manage at the front   | API Gateway                               |
| One service abusing another                | Rate Limiting                             |
| Need to track everything across services   | Centralised logging + distributed tracing |

---

## Key Numbers Every Designer Should Know

These rough numbers help you make quick decisions:

| Operation                                 | Time    |
| ----------------------------------------- | ------- |
| L1 cache reference                        | ~0.5 ns |
| L2 cache reference                        | ~7 ns   |
| Main memory (RAM) access                  | ~100 ns |
| SSD random read                           | ~150 Âµs |
| Network round trip within same datacenter | ~0.5 ms |
| HDD seek                                  | ~10 ms  |
| Network round trip cross-continent        | ~150 ms |

> **Rule of thumb:** RAM is 1000x faster than SSD. SSD is 100x faster than HDD. Anything over the network is "slow."

---

## Glossary

| Term                        | Plain English                                                             |
| --------------------------- | ------------------------------------------------------------------------- |
| **Latency**                 | How long does one request take?                                           |
| **Throughput**              | How many requests can you handle per second?                              |
| **Horizontal scaling**      | Add more machines                                                         |
| **Vertical scaling**        | Make one machine bigger                                                   |
| **Stateless**               | Server doesn't remember you between requests (enables horizontal scaling) |
| **Idempotent**              | Doing the same operation twice gives the same result (safe to retry)      |
| **SLA**                     | Promise to the customer about uptime (e.g. 99.9%)                         |
| **Hot spot**                | One shard/server getting way more load than others                        |
| **Single point of failure** | One thing whose failure brings everything down                            |
| **Fanout**                  | One event triggering writes to many places                                |
| **Eventual consistency**    | Data will be consistent... but maybe not immediately                      |

---

_Part of the AI_Projects learning series._