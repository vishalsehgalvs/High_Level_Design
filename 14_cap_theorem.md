# 14 — CAP Theorem: The Impossible Triangle

## The Big Idea

In any distributed system — any system where data lives on more than one machine — you can only guarantee **two out of these three** properties at the same time:

- **C**onsistency
- **A**vailability
- **P**artition Tolerance

This is the **CAP Theorem**, proven by Eric Brewer in 2000.

```mermaid
graph TD
    C[Consistency - Every read gets the most recent write]
    A[Availability - Every request gets a response]
    P[Partition Tolerance - System works even when network splits]

    C --- A
    A --- P
    P --- C
    
    Center[Pick any 2]
    C -.- Center
    A -.- Center
    P -.- Center
```

---

## What Each Property Means

### Consistency (C)

Every node in the system sees the **same data at the same time**. If you write a value on Node 1, any read from Node 2 immediately sees that new value.

```mermaid
sequenceDiagram
    participant NodeA
    participant NodeB

    Note over NodeA,NodeB: Consistent system
    NodeA->>NodeA: Write: x = 5
    NodeA->>NodeB: Sync: x = 5
    Note over NodeB: Read x → 5 ✓ correct
```

*Not to be confused with ACID consistency — this is about all nodes agreeing on the same value.*

---

### Availability (A)

**Every request receives a response** — it won't just hang or return an error. The response might not have the absolute latest data, but you'll get *some* response.

```mermaid
sequenceDiagram
    participant Client
    participant Node

    Client->>Node: Read x
    Node-->>Client: Returns x = 4 (might be slightly old, but responds)
```

---

### Partition Tolerance (P)

The system **continues operating** even if the network between nodes breaks and nodes can't talk to each other.

```mermaid
flowchart LR
    N1[(Node 1)] ---|❌ Network split!| N2[(Node 2)]
    
    Client1[Client] -->|still works| N1
    Client2[Client] -->|still works| N2
```

A network partition is when some nodes can't communicate with others. In any real distributed system (especially those spread across data centers), this **will happen**. So partition tolerance is essentially non-negotiable.

---

## The Real Choice: CP vs AP

Since partition tolerance is required, you're really choosing between:

```mermaid
graph LR
    CP[CP - Consistent and Partition Tolerant]
    AP[AP - Available and Partition Tolerant]

    CP --> c1[When a partition happens, stop serving requests rather than return stale data]
    CP --> c2[Prioritizes correctness over availability]
    CP --> c3[Examples: HBase, Zookeeper, MongoDB by default, banks]

    AP --> a1[When a partition happens, keep serving requests, even with possibly stale data]
    AP --> a2[Prioritizes uptime over correctness]
    AP --> a3[Examples: Cassandra, CouchDB, DynamoDB, DNS]
```

---

## Real-World Examples

### Bank Transfer: CP System

When you transfer money from Account A to Account B:
- Account A must decrease
- Account B must increase
- If there's a network partition, the bank **stops processing transfers** until it's resolved

```mermaid
sequenceDiagram
    participant User
    participant BankA as Node A (US)
    participant BankB as Node B (EU)

    Note over BankA,BankB: Network partition!
    User->>BankA: Transfer $100 to Bob
    BankA->>BankB: Can't reach you...
    BankA-->>User: ❌ Cannot process right now - system unavailable

    Note over BankA,BankB: Partition healed
    BankA->>BankB: Now transferring
    BankA-->>User: ✅ Transfer complete
```

Better to reject the request than to accidentally create or destroy money.

---

### Social Media Feed: AP System

When you post a photo, it's okay if some users see it 2 seconds later than others. Eventual consistency is fine here.

```mermaid
sequenceDiagram
    participant User
    participant NodeUS as Node US
    participant NodeEU as Node EU

    Note over NodeUS,NodeEU: Network partition!
    User->>NodeUS: Post photo
    NodeUS-->>User: ✅ Posted!
    
    Note over NodeEU: Has stale data for a few seconds
    Someone->>NodeEU: View feed
    NodeEU-->>Someone: Returns feed without the new photo (slightly stale)

    Note over NodeUS,NodeEU: Partition healed - eventually consistent
    NodeUS->>NodeEU: Sync new photo
    NodeEU->>NodeEU: Feed now up to date
```

Nobody is harmed if a photo appears 2 seconds late. The system stays available.

---

## Consistency Models (Beyond Just "Consistent")

Consistency isn't binary. There's a spectrum:

```mermaid
graph LR
    Strong[Strong Consistency] -->|weaker| Sequential[Sequential Consistency]
    Sequential -->|weaker| Causal[Causal Consistency]
    Causal -->|weaker| Eventual[Eventual Consistency]

    Strong --> s1[Every read sees the absolute latest write]
    Sequential --> s2[All nodes see operations in the same order]
    Causal --> c1[Operations that are causally related are seen in order]
    Eventual --> e1[Given enough time with no new writes, all nodes will converge]
```

| Model | Example |
|---|---|
| **Strong** | Reading your balance at a bank ATM |
| **Sequential** | Google Docs with real-time sync |
| **Causal** | Replies to a tweet must appear after the original tweet |
| **Eventual** | DNS updates, shopping cart sync |

---

## Summary Decision Guide

```mermaid
flowchart TD
    Q1{Is wrong data worse than no response?}
    Q1 -->|Yes - e.g. banking, inventory| CP[Choose CP System]
    Q1 -->|No - e.g. social feed, recommendations| AP[Choose AP System]
    
    CP --> CPEx[HBase, MongoDB, PostgreSQL with sync replication]
    AP --> APEx[Cassandra, DynamoDB, CouchDB, DNS]
```

| Scenario | Best choice | Why |
|---|---|---|
| Bank transfers | CP | Wrong answer is catastrophic |
| Social media feed | AP | Slightly stale is fine |
| Shopping cart | AP (usually) | Amazon famously chose AP for carts |
| User authentication | CP | Must validate against latest password/token |
| Search suggestions | AP | A stale suggestion is fine |
| Stock trading | CP | Price must be correct |
