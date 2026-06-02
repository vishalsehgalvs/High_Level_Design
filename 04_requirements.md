# 04 — Functional vs Non-Functional Requirements

## Why Requirements Come First

You can't design a system if you don't know what it's supposed to do. Before drawing a single box, you clarify **two types of requirements**:

1. **Functional** — what the system *does*
2. **Non-Functional** — how *well* it does it

```mermaid
flowchart LR
    Requirements --> FR[Functional Requirements]
    Requirements --> NFR[Non-Functional Requirements]
    
    FR --> f1[Features the user experiences]
    FR --> f2[What buttons do what]
    FR --> f3[What data is stored]
    
    NFR --> n1[How fast?]
    NFR --> n2[How reliable?]
    NFR --> n3[How many users?]
    NFR --> n4[How secure?]
```

---

## Functional Requirements

These are the features. The *what*.

**Example: Design a messaging app**

| Functional Requirement | Description |
|---|---|
| Users can register and log in | Auth system needed |
| Users can send text messages | Core feature |
| Users can see message history | Read past messages |
| Users can create group chats | Group functionality |
| Messages show delivery and read receipts | Status tracking |

Functional requirements are usually straightforward — they come from the product spec or user stories.

---

## Non-Functional Requirements

These are the quality attributes. The *how well*. This is where system design lives.

---

### 1. Scalability

Can the system handle growth without breaking?

```mermaid
graph LR
    S[Scalability] --> Vertical[Vertical - Bigger machine]
    S --> Horizontal[Horizontal - More machines]
    
    Vertical --> v1[Upgrade RAM, CPU, disk]
    Vertical --> v2[Simpler to implement]
    Vertical --> v3[Has a ceiling - you can only get so big]
    
    Horizontal --> h1[Add more servers]
    Horizontal --> h2[More complex - need load balancers]
    Horizontal --> h3[Theoretically unlimited scale]
```

*"If we get 10x the users tomorrow, does the system survive?"*

---

### 2. Availability

What % of the time is the system up and running?

| Availability | Downtime per year | Used for |
|---|---|---|
| 99% | 3.65 days | Internal tools |
| 99.9% | 8.7 hours | Most web apps |
| 99.99% | 52 minutes | E-commerce, payments |
| 99.999% ("five nines") | 5 minutes | Banking, hospitals |

```mermaid
flowchart LR
    A[High Availability] --> R1[No single point of failure]
    A --> R2[Redundant servers]
    A --> R3[Automatic failover]
    A --> R4[Health checks]
    A --> R5[Multi-region deployment]
```

*"If one server dies at 3am, do users even notice?"*

---

### 3. Reliability

Will the system produce the correct result every time?

Availability ≠ Reliability. A system can be:
- Available but unreliable (it's up, but gives wrong answers)
- Reliable but not highly available (it's correct when up, but goes down sometimes)

```mermaid
graph TD
    Reliability --> Correct[Correct results always]
    Reliability --> Durable[Data never gets lost]
    Reliability --> Idempotent[Same operation twice = same result]
```

*"If we process a bank transfer, will it happen exactly once — not zero times, not twice?"*

---

### 4. Performance

How fast is the system?

Two key metrics:

| Metric | What it measures | Example |
|---|---|---|
| **Latency** | Time for one request to complete | Page loads in 200ms |
| **Throughput** | How many requests can be handled per second | 10,000 requests/second |

```mermaid
flowchart LR
    Request -->|latency: 100ms| Response
    
    subgraph "Throughput"
        R1[Req 1] & R2[Req 2] & R3[Req 3] -->|1000 per second| Server
    end
```

*"The Google Search home page loads in under 200 milliseconds. How?"*

---

### 5. Security

Can attackers steal data, crash the system, or impersonate users?

```mermaid
graph TD
    Security --> Auth[Authentication - Are you who you say you are?]
    Security --> Authz[Authorization - Are you allowed to do this?]
    Security --> Enc[Encryption - Data safe in transit and at rest]
    Security --> Input[Input validation - No SQL injection etc.]
    Security --> Rate[Rate limiting - No brute force attacks]
    Security --> Audit[Audit logs - Who did what and when?]
```

---

### 6. Maintainability

Can engineers add features, fix bugs, and deploy changes without everything breaking?

```mermaid
graph LR
    Maintainability --> Clean[Clean code and docs]
    Maintainability --> Tests[Test coverage]
    Maintainability --> CI[CI/CD pipelines]
    Maintainability --> Modular[Modular architecture - change one thing without breaking others]
    Maintainability --> Rollback[Easy rollback when deployment goes wrong]
```

---

### 7. Observability

Can you tell what's happening inside the system at any moment?

The three pillars of observability:

```mermaid
graph TD
    O[Observability] --> Metrics[Metrics - numbers over time]
    O --> Logs[Logs - text records of events]
    O --> Traces[Traces - follow one request through all services]
    
    Metrics --> m1[CPU usage, request rate, error %]
    Logs --> l1[Error messages, access logs]
    Traces --> t1[See exactly where a slow request got stuck]
```

---

## The Full Picture

```mermaid
mindmap
    root(System Requirements)
        Functional
            User features
            Business logic
            Data operations
        Non-Functional
            Scalability
            Availability
            Reliability
            Performance
            Security
            Maintainability
            Observability
```

## How to Gather Requirements in an Interview

1. **Clarify scope** — "Are we designing just the backend? Including auth? Mobile app too?"
2. **Estimate scale** — "How many daily active users? Requests per second?"
3. **Identify priorities** — "Is consistency more important than availability here?"
4. **Ask about constraints** — "What's the budget? Existing tech stack?"

Then write down your requirements clearly before you start designing.
