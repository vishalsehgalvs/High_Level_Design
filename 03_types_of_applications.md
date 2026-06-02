# 03 — Types of Applications: Data-Intensive vs Compute-Intensive

## The Key Question Before Designing Anything

Before you draw a single box in your architecture diagram, ask: **"Where is the bottleneck going to be?"**

- Is the problem about **moving and storing a lot of data**?
- Or is the problem about **doing a lot of computation on that data**?

The answer changes everything about how you design the system.

---

## Data-Intensive Applications

A **data-intensive** application's main challenge is:
- Storing huge amounts of data
- Reading and writing it fast
- Keeping it consistent and available
- Moving it efficiently between services

The bottleneck is **I/O** — disk reads, network transfers, database queries.

```mermaid
flowchart LR
    Input[Huge volumes of data] --> Store[(Storage)]
    Store -->|fast reads| Query[Query Engine]
    Query --> Output[Processed results]

    style Input fill:#ff9999
    style Store fill:#ff9999
    style Query fill:#ff9999
```

### Real-world examples

| Application | Why it's data-intensive |
|---|---|
| Instagram | 100M+ photos uploaded per day, billions stored |
| WhatsApp | Billions of messages per day |
| Twitter/X | 500M tweets/day, complex timeline generation |
| Uber | Real-time GPS data from millions of drivers |
| E-commerce | Huge product catalogs, order histories, search indexes |

### Instagram Architecture — a data-intensive system

```mermaid
flowchart TD
    App[Instagram App] --> LB[Load Balancer]
    LB --> API[API Servers]
    API --> Cache[(Redis - user feeds, sessions)]
    Cache -->|miss| DB[(PostgreSQL - user data)]
    API --> S3[(AWS S3 - photos and videos)]
    API --> CDN[CDN - fast global delivery of media]
    API --> Search[Elasticsearch - search index]
    DB --> Replica1[(Read Replica - Asia)]
    DB --> Replica2[(Read Replica - US)]
```

The main challenge here isn't complex math — it's handling the **volume** of data and serving it fast to billions of people worldwide.

---

## Compute-Intensive Applications

A **compute-intensive** application's main challenge is:
- Doing heavy mathematical operations
- Processing data to produce results
- The CPU/GPU is the bottleneck, not storage

```mermaid
flowchart LR
    Input[Input data] --> CPU[Heavy computation - CPU/GPU]
    CPU -->|many cycles later| Output[Result]

    style CPU fill:#9999ff
```

### Real-world examples

| Application | Why it's compute-intensive |
|---|---|
| Netflix video encoding | Converts raw video into 10+ formats and resolutions |
| YouTube video processing | Same — transcoding is CPU-heavy |
| ML model training | Billions of matrix multiplications |
| Real-time fraud detection | ML inference on every transaction |
| Scientific simulations | Weather modeling, protein folding |
| Rendering (games/VFX) | Billions of pixel calculations |

### Video encoding pipeline (YouTube-style)

```mermaid
flowchart TD
    Upload[User uploads raw video - 4K 60fps] --> Queue[Message Queue]
    Queue --> Worker1[Transcoding Worker - 4K]
    Queue --> Worker2[Transcoding Worker - 1080p]
    Queue --> Worker3[Transcoding Worker - 720p]
    Queue --> Worker4[Transcoding Worker - 480p]
    Worker1 & Worker2 & Worker3 & Worker4 --> Storage[(CDN / Object Storage)]
    Storage --> User[Streamed to users]
```

The bottleneck here is **CPU time** — raw video transcoding is one of the most compute-heavy tasks in software.

---

## CPU-Bound vs GPU-Bound

```mermaid
graph LR
    Compute[Compute-Intensive] --> CPU[CPU-Bound]
    Compute --> GPU[GPU-Bound]

    CPU --> c1[Sequential logic]
    CPU --> c2[Complex decision trees]
    CPU --> c3[Database query execution]
    CPU --> c4[Web server request handling]

    GPU --> g1[Machine learning training]
    GPU --> g2[Image/video processing]
    GPU --> g3[3D rendering]
    GPU --> g4[Large matrix operations]
```

A GPU has thousands of simpler cores designed to do the **same operation on many pieces of data simultaneously** (parallel processing). A CPU has a few powerful cores great for complex sequential logic.

---

## Why This Matters for System Design

| Aspect | Data-Intensive | Compute-Intensive |
|---|---|---|
| Main bottleneck | Disk I/O, network, database | CPU cycles, GPU memory |
| Scale strategy | More replicas, better caching, sharding | More powerful machines, GPU clusters |
| Cost driver | Storage, bandwidth | Compute time, specialized hardware |
| Latency concern | Database query time, cache miss | Processing time per job |
| Example fix | Add a read replica | Add more transcoding workers |

---

## Real-world Scalability Concerns

### Instagram — a mix of both

```mermaid
flowchart LR
    Feed[Feed Generation] -->|data-intensive| ReadDB[Read replicas + cache]
    Filter[Face Filter / AR] -->|compute-intensive| GPU[GPU on device or edge]
    Upload[Photo Upload] -->|data-intensive| S3[Object storage]
    Recommendation[Who to follow?] -->|compute-intensive| MLModel[ML recommendation model]
```

Most large systems are **hybrids** — the feed is data-intensive, but the ML recommendation engine is compute-intensive. You design each part of the system for its own bottleneck.

---

## The Simple Mental Check

Before designing any component, ask yourself:

```mermaid
flowchart TD
    Q[What is the bottleneck?] --> A{More data or more math?}
    A -->|More data| DI[Data-intensive approach]
    A -->|More math| CI[Compute-intensive approach]
    DI --> d1[Better databases]
    DI --> d2[Caching layers]
    DI --> d3[Data replication]
    DI --> d4[Efficient data formats]
    CI --> c1[More CPU/GPU power]
    CI --> c2[Background job workers]
    CI --> c3[Async processing queues]
    CI --> c4[Distributed computation]
```
