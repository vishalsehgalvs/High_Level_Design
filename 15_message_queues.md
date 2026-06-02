# 15 — Message Queues: Don't Do Heavy Work Right Now

## The Core Idea

When a user does something — uploads a photo, places an order, sends an email — you don't have to do *everything* related to that action immediately before responding to them.

A **message queue** is a buffer between producers (things that create work) and consumers (things that process that work).

```mermaid
flowchart LR
    Producer[API Server - Producer] -->|Adds job to queue| Queue[(Message Queue)]
    Queue -->|Delivers job when ready| C1[Worker 1]
    Queue -->|Delivers job when ready| C2[Worker 2]
    Queue -->|Delivers job when ready| C3[Worker 3]
```

**Without a queue:**
- User uploads photo
- Server resizes it → 2 seconds
- Server generates thumbnails → 1 second
- Server sends notification email → 1 second
- Server updates user's feed → 500ms
- User waits **4.5 seconds** for a response

**With a queue:**
- User uploads photo
- Server saves the raw photo → 100ms
- Server adds jobs to queue → instant
- User gets response: **"Photo uploaded!"** in 100ms
- Workers process resizing, thumbnails, emails in the background

---

## Synchronous vs Asynchronous Processing

```mermaid
flowchart TD
    subgraph "Synchronous - User waits for everything"
        U1[User] -->|request| A1[API]
        A1 -->|does everything right now| Task1[Resize image]
        Task1 --> Task2[Send email]
        Task2 --> Task3[Update search index]
        Task3 -->|done - 5 seconds later| U1
    end

    subgraph "Asynchronous - User gets quick response"
        U2[User] -->|request| A2[API]
        A2 -->|stores job| Q[(Queue)]
        A2 -->|immediate response - 100ms| U2
        Q --> W1[Worker: Resize image]
        Q --> W2[Worker: Send email]
        Q --> W3[Worker: Update search index]
    end
```

---

## FIFO Queue (First In, First Out)

The simplest kind of queue. Jobs are processed in the order they arrive. First job in → first job processed.

```mermaid
flowchart LR
    J1[Job 1 - first in] --> Q[(Queue)]
    J2[Job 2] --> Q
    J3[Job 3] --> Q
    Q -->|Job 1 first out| W[Worker]
```

**Used for:** Order processing, basic task queues where order matters

---

## Priority Queue

Some jobs are more urgent than others. A priority queue processes high-priority jobs first, regardless of when they arrived.

```mermaid
flowchart TD
    Q[(Priority Queue)]
    J_Low[Low priority job - arrives first] --> Q
    J_High[High priority job - arrives second] --> Q
    Q -->|Processes high priority first| W[Worker]
```

**Real example:**
- **Priority 1 (urgent)**: password reset email — send immediately
- **Priority 2 (normal)**: weekly digest email — can wait
- **Priority 3 (low)**: analytics batch job — send whenever

---

## Pub/Sub (Publish-Subscribe)

In a regular queue, each message goes to **one consumer**. In pub/sub, one message goes to **many consumers** (subscribers).

```mermaid
flowchart LR
    Publisher[Publisher - Order Service] -->|"Order #42 placed"| Topic[(Topic: orders)]
    Topic -->|Notifies| Sub1[Subscriber: Inventory Service]
    Topic -->|Notifies| Sub2[Subscriber: Email Service]
    Topic -->|Notifies| Sub3[Subscriber: Analytics Service]
    Topic -->|Notifies| Sub4[Subscriber: Fraud Detection]
```

Each subscriber gets the same event and processes it independently.

**Used for:** Event-driven systems, microservices that react to the same event

**Examples:** Apache Kafka, Google Pub/Sub, AWS SNS + SQS

---

## Push vs Pull Model

### Push (Server pushes to consumers)

The queue actively sends messages to consumers.

```mermaid
sequenceDiagram
    Queue->>Worker: Here's a job, process it!
    Worker-->>Queue: Done
```

- Workers don't need to poll
- Risk: if the worker is slow, the queue keeps pushing and overwhelms it

---

### Pull (Consumers pull from queue)

Workers ask the queue for work when they're ready.

```mermaid
sequenceDiagram
    Worker->>Queue: Give me a job
    Queue-->>Worker: Here's one
    Worker->>Worker: Process it
    Worker->>Queue: Done. Give me another
```

- Worker controls its own rate
- Better for slow or variable-speed workers
- Most modern systems (SQS, Kafka consumers) use pull

---

## Poison Messages

A **poison message** is a message that causes a worker to crash every time it tries to process it. Without protection, the message keeps re-queuing and crashing workers in a loop.

```mermaid
flowchart TD
    Q[(Queue)] --> W1[Worker - crashes on this message 💥]
    W1 -->|Message returned to queue| Q
    Q --> W2[Worker - crashes again 💥]
    W2 -->|Message returned to queue| Q
    Q --> Repeat[...infinite loop]
```

**Solution: Dead Letter Queue (DLQ)**

After a message fails N times (e.g., 3), move it to a **dead letter queue** instead of retrying forever.

```mermaid
flowchart LR
    Q[(Main Queue)] -->|Fails 3 times| DLQ[(Dead Letter Queue)]
    DLQ --> Alert[Alert engineer to investigate]
    DLQ --> Replay[After fix, replay message]
```

---

## Duplicate Messages

In distributed systems, messages can sometimes be delivered **more than once** (e.g., network hiccup causes the worker to receive the same job twice).

**At-least-once delivery**: guaranteed to be delivered but may deliver twice.

**Solution: Idempotency**

Design your consumers to be **idempotent** — processing the same message twice produces the same result as processing it once.

```mermaid
flowchart LR
    M["Job: send email to user@email.com (job_id: 789)"] --> Worker
    Worker -->|Check: was job 789 already done?| DB[(Processed jobs DB)]
    DB -->|Yes, already done| Skip[Skip - do nothing]
    DB -->|No| Process[Send email and mark job 789 as done]
```

---

## Real Systems at a Glance

| System | Type | Used for |
|---|---|---|
| **Apache Kafka** | Distributed log / pub-sub | High-throughput event streaming |
| **RabbitMQ** | Traditional queue + pub-sub | Microservice messaging |
| **AWS SQS** | Managed FIFO / standard queue | Cloud-native queuing |
| **AWS SNS** | Pub-sub | Fan-out notifications |
| **Redis Streams** | Lightweight stream queue | Small scale, fast |
| **Google Pub/Sub** | Managed pub-sub | GCP event-driven systems |

---

## Full Picture: Order Processing System

```mermaid
flowchart TD
    User[User places order] --> API[Order API]
    API -->|Saves order to DB| DB[(Orders DB)]
    API -->|Publishes: order.placed| Kafka[(Kafka Topic: order.placed)]
    API -->|Returns 201 immediately| User

    Kafka -->|Subscribes| Inventory[Inventory Service - reserve stock]
    Kafka -->|Subscribes| Email[Email Service - confirmation email]
    Kafka -->|Subscribes| Fraud[Fraud Detection Service]
    Kafka -->|Subscribes| Analytics[Analytics Service]

    Fraud -->|Suspicious?| FraudQueue[(Fraud Review Queue)]
    FraudQueue --> FraudWorker[Manual Review Worker]
```
