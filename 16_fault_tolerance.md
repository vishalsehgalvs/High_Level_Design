# 16 — Fault Tolerance: Building Systems That Don't Give Up

## What Is Fault Tolerance?

A **fault** is anything that goes wrong — a hard drive crashing, a server freezing, a developer pushing bad code, a network cable being unplugged.

**Fault tolerance** means the system keeps working (or degrades gracefully) even when faults happen.

The key insight: **you don't prevent failures — you design for them**.

```mermaid
flowchart LR
    World[Real World] -->|always produces| Faults[Faults and Failures]
    Faults -->|naive system| Down[System is down]
    Faults -->|fault-tolerant system| Running[System keeps running]
```

---

## Types of Failures

### 1. Hardware Failures

```mermaid
mindmap
    root((Hardware Failures))
        Disk crash
            Data loss if not replicated
        Server dies unexpectedly
            Service unavailable
        RAM corruption
            Data corruption or crash
        NIC failure
            Network disconnected
        Data center power outage
            Entire region goes offline
```

**Solutions:**
- Replicate data across multiple disks (RAID) and servers
- Deploy across multiple availability zones / regions
- Have standby servers ready for immediate failover

---

### 2. Software Failures

Software failures are sneakier than hardware ones because they often affect **all instances** equally.

```mermaid
flowchart TD
    subgraph "Common Software Failures"
        MemoryLeak[Memory Leak - app uses more RAM over time until it crashes]
        Bug[Bug in code - e.g. divide by zero on a specific input]
        Cascading[Cascading failure - one slow service causes all others to queue up and die]
        ResourceExhaustion[Thread / connection pool exhaustion]
    end
```

**Cascading failure example:**

```mermaid
flowchart TD
    Gateway[API Gateway] -->|calls| UserService[User Service]
    UserService -->|calls| RecoService[Recommendation Service]
    RecoService -->|calls| SlowDB[(Slow Database - 30s queries)]
    
    SlowDB -->|Response backed up| RecoService
    RecoService -->|All threads waiting| UserService
    UserService -->|All threads waiting| Gateway
    Gateway -->|All threads waiting| Down[Entire system unresponsive 💥]
```

One slow database → entire platform dies.

---

### 3. Human Errors

Studies show **human errors cause more outages than hardware failures**.

```mermaid
mindmap
    root((Human Errors))
        Wrong config pushed to production
            Typo in config file causes crash
        Accidental data deletion
            DROP TABLE by mistake
        Capacity miscalculation
            Not enough servers for big event
        Bad deployment
            New code breaks in production
        Security misconfiguration
            S3 bucket made public accidentally
```

**Solutions:**
- Blue/Green deployments — test in a mirror environment before switching traffic
- Canary releases — roll out to 1% of users first
- Rollback capability — quickly revert to the last known good version
- Change management — require review/approval before config changes
- Backups with tested restore procedures

---

## Recovery Strategies

### Retry with Exponential Backoff

When a call fails, don't retry immediately — you'll hammer an already-struggling service. Wait a bit. Wait longer next time.

```mermaid
flowchart TD
    Call[Try request] -->|Fails| W1[Wait 1 second]
    W1 --> Retry1[Retry] -->|Fails| W2[Wait 2 seconds]
    W2 --> Retry2[Retry] -->|Fails| W3[Wait 4 seconds]
    W3 --> Retry3[Retry] -->|Fails| W4[Wait 8 seconds]
    W4 --> Retry4[Final attempt] -->|Fails| Err[Return error to user]
```

**Jitter**: Add a small random delay so all clients don't retry at the exact same time (which would cause a thundering herd problem).

---

### Circuit Breaker Pattern

Inspired by electrical circuits. If a service is failing, **stop sending it requests** for a while to let it recover.

```mermaid
stateDiagram-v2
    [*] --> Closed : Normal operation
    Closed --> Open : Error threshold exceeded (e.g. 50% failures in 10s)
    Open --> HalfOpen : Wait period expires (e.g. 30 seconds)
    HalfOpen --> Closed : Test request succeeds
    HalfOpen --> Open : Test request fails
```

**State meanings:**
- **Closed (green)**: Everything normal, requests flow through
- **Open (red)**: Too many failures — immediately return error without even trying the downstream service
- **Half-Open**: Let one request through as a test — if it succeeds, close the circuit again

```mermaid
sequenceDiagram
    participant Client
    participant CircuitBreaker as Circuit Breaker
    participant Service

    Note over CircuitBreaker: State: Closed
    Client->>CircuitBreaker: Request
    CircuitBreaker->>Service: Forward request
    Service-->>CircuitBreaker: Error!
    
    Note over CircuitBreaker: Too many errors - State: Open
    Client->>CircuitBreaker: Request
    CircuitBreaker-->>Client: ❌ Fast fail (no call to service)

    Note over CircuitBreaker: 30 seconds later - State: Half-Open
    Client->>CircuitBreaker: Request
    CircuitBreaker->>Service: Test request
    Service-->>CircuitBreaker: ✅ Success
    Note over CircuitBreaker: State: Closed again
```

**Benefits:**
- Fails fast instead of making the client wait for a timeout
- Prevents cascading failures
- Gives the failing service breathing room

---

### Redundancy

Have more than you need so that if something fails, a spare is ready.

```mermaid
flowchart LR
    LB[Load Balancer] --> S1[Server 1 - Active]
    LB --> S2[Server 2 - Active]
    LB --> S3[Server 3 - Standby]

    S1 -->|S1 dies| S3
    S3 -->|Activates| Active[Now serving traffic]
```

**Types:**

| Type | Description | Example |
|---|---|---|
| **Active-Active** | Multiple instances all serving traffic | Multiple web servers behind a load balancer |
| **Active-Passive** | Backup waits in standby, takes over on failure | Database primary/replica failover |
| **Geographic redundancy** | Copies in multiple regions | AWS us-east-1 and us-west-2 |

---

### Bulkhead Pattern

Isolate different parts of the system so a failure in one doesn't sink everything else.

Named after the compartments on a ship — if one compartment floods, the ship doesn't sink.

```mermaid
flowchart TD
    subgraph "Without Bulkhead"
        Shared[Shared Thread Pool - 100 threads]
        SlowFeature[Slow feature - uses all 100 threads]
        AllFeatures[All other features starved 💥]
    end

    subgraph "With Bulkhead"
        Video[Video service - 30 threads]
        Search[Search - 30 threads]
        Auth[Auth - 20 threads]
        Other[Other - 20 threads]
        Video -->|Video is slow| VideoOnly[Only video is degraded]
        Search --> SearchOK[Search still works ✓]
        Auth --> AuthOK[Auth still works ✓]
    end
```

---

## Key Metrics for Fault Tolerance

| Metric | What it measures |
|---|---|
| **MTBF** (Mean Time Between Failures) | Average time between failures — higher is better |
| **MTTR** (Mean Time To Recovery) | Average time to recover from a failure — lower is better |
| **RTO** (Recovery Time Objective) | Maximum acceptable downtime during an outage |
| **RPO** (Recovery Point Objective) | Maximum acceptable amount of data loss |

```mermaid
flowchart LR
    Normal[Normal operation] -->|Failure occurs| Down[System down]
    Down -->|Detected and fixed| Recover[System restored]
    
    Normal --> MTBF[MTBF: time between this and last failure]
    Down --> MTTR[MTTR: time to recover]
    Recover --> RPO[RPO: how much data was lost?]
```

**Example:** "Our RTO is 1 hour and RPO is 10 minutes" means:
- The system must be back up within 1 hour of an outage
- We cannot lose more than 10 minutes of data
