# 12 — Replication: Keeping Your Data Safe and Available

## The Problem

If you have one database and it dies, your entire system dies with it. All your users' data is gone. Your app is down. This is called a **single point of failure**.

**Replication** means keeping **copies of the same data on multiple machines**. If one machine dies, others still have the data.

```mermaid
flowchart LR
    subgraph "Without Replication"
        App1[App] --> DB_Single[(Single DB)]
        DB_Single --> Crash[💥 DB crashes = everything is gone]
    end

    subgraph "With Replication"
        App2[App] --> Primary[(Primary DB)]
        Primary -->|replicates to| R1[(Replica 1)]
        Primary -->|replicates to| R2[(Replica 2)]
        Primary -->|crashes| Failover[Replica 1 takes over]
    end
```

Replication gives you:
- **High availability** — the system stays up even if some nodes die
- **Read scalability** — multiple replicas can serve read requests
- **Disaster recovery** — data is safe even if a data center burns down

---

## Replication Algorithms

### 1. Single Leader Replication (Primary-Replica)

One node is the **leader** (primary). All writes go to the leader. The leader sends changes to **followers** (replicas). Reads can come from any node.

```mermaid
flowchart TD
    App[Application]
    App -->|All WRITES| Leader[(Leader / Primary)]
    Leader -->|Replicates changes| F1[(Follower 1)]
    Leader -->|Replicates changes| F2[(Follower 2)]
    Leader -->|Replicates changes| F3[(Follower 3 - in another region)]
    App -->|READs can go to any| F1
    App -->|READs can go to any| F2
    App -->|READs can go to any| Leader
```

**Failure scenario:**
```mermaid
flowchart TD
    Leader[(Leader - CRASHES)] --> Election[Followers hold election]
    Election --> NewLeader[(Follower 1 becomes new leader)]
    NewLeader -->|Starts accepting writes| App[Application]
```

**Used by:** MySQL, PostgreSQL, MongoDB

**Pros:** Simple, easy to reason about consistency
**Cons:** Leader is a write bottleneck; failover takes time

---

### 2. Multi-Leader Replication

Multiple nodes can accept **writes**. Each leader replicates to others.

```mermaid
flowchart LR
    LeaderUS[(Leader - US)] -->|replicate| LeaderEU[(Leader - EU)]
    LeaderEU -->|replicate| LeaderAS[(Leader - Asia)]
    LeaderAS -->|replicate| LeaderUS
    
    UserUS[US Users] -->|write locally| LeaderUS
    UserEU[EU Users] -->|write locally| LeaderEU
    UserAS[Asian Users] -->|write locally| LeaderAS
```

**Pros:** Lower write latency (write to your local region)
**Cons:** **Conflict resolution is hard** — what if two users edit the same document from different regions simultaneously?

**Used by:** Google Docs (offline editing), Notion, distributed databases like CouchDB

---

### 3. Leaderless Replication

No single leader. Any node can accept writes. The client writes to multiple nodes and reads from multiple nodes.

```mermaid
flowchart TD
    Client[Client]
    Client -->|write to 3 out of 5 nodes| N1[(Node 1 ✓)]
    Client -->|write to 3 out of 5 nodes| N2[(Node 2 ✓)]
    Client -->|write to 3 out of 5 nodes| N3[(Node 3 ✓)]
    N4[(Node 4 - missed write)]
    N5[(Node 5 - missed write)]
    
    Client -->|read from 3 nodes, take latest| N1
    Client -->|read from 3 nodes| N3
    Client -->|read from 3 nodes| N4
```

Quorum ensures consistency: if you write to W nodes and read from R nodes, and W + R > total nodes, you're guaranteed to read at least one node that has the latest write.

**Used by:** Amazon DynamoDB, Apache Cassandra

---

## Synchronous vs Asynchronous Replication

```mermaid
sequenceDiagram
    participant App
    participant Leader as Leader (Primary)
    participant Replica

    Note over App,Replica: SYNCHRONOUS
    App->>Leader: Write
    Leader->>Replica: Replicate
    Replica-->>Leader: Ack
    Leader-->>App: Success (only after replica confirms)

    Note over App,Replica: ASYNCHRONOUS
    App->>Leader: Write
    Leader-->>App: Success (immediately)
    Leader->>Replica: Replicate (in background, later)
```

| | Synchronous | Asynchronous |
|---|---|---|
| **Write speed** | Slower (waits for replica) | Faster (doesn't wait) |
| **Data safety** | Stronger (confirmed on multiple nodes) | Weaker (could lose recent writes if leader crashes) |
| **Availability** | Lower (if replica is slow, writes slow down) | Higher (leader doesn't wait) |
| **Common use** | Financial transactions | Most web applications |

---

## Quorum

Quorum is the minimum number of nodes that must agree for an operation to succeed.

Given:
- **N** = total number of replicas
- **W** = minimum nodes that must confirm a write
- **R** = minimum nodes that must respond to a read

**Rule for strong consistency:** W + R > N

```mermaid
flowchart TD
    Q[5 nodes total - N=5]
    Q --> Write[Write quorum W=3 - 3 nodes must confirm write]
    Q --> Read[Read quorum R=3 - 3 nodes must respond to read]
    
    Q --> Check["W + R = 6 > N = 5 ✓ Strong consistency guaranteed"]
```

**Example (N=5, W=3, R=3):**
- A write succeeds when 3 out of 5 nodes confirm it
- A read fetches from 3 nodes and returns the most recent version
- Since W + R > N, the read set and write set overlap by at least 1 node — so the read always sees the latest write

**Tuning quorum for different needs:**

| Configuration | Effect |
|---|---|
| W=1, R=5 | Fast writes, slow reads |
| W=5, R=1 | Slow writes, fast reads |
| W=3, R=3 (N=5) | Balanced |
| W=1, R=1 | Fast everything — but no consistency guarantees |
