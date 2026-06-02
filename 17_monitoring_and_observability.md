# 17 — Monitoring and Observability: Knowing What's Happening

## The Difference

- **Monitoring**: You know what to watch → you set up dashboards and alerts for known problems
- **Observability**: You can **figure out what's wrong** even for problems you didn't anticipate

A system is observable when you can ask it any question about its internal state just from its external outputs — without modifying the code.

```mermaid
flowchart LR
    subgraph "Your System"
        Services[Microservices, Databases, Queues]
    end

    Services -->|Emit| Metrics[Metrics - numbers]
    Services -->|Emit| Logs[Logs - text events]
    Services -->|Emit| Traces[Traces - request journeys]

    Metrics --> Dashboard[Grafana / Datadog Dashboard]
    Logs --> LogSearch[Log search - Elasticsearch / CloudWatch]
    Traces --> TraceViewer[Trace viewer - Jaeger / Zipkin]

    Dashboard --> Alerts[Alerts and Pager]
```

---

## The Three Pillars of Observability

### Pillar 1: Metrics

Metrics are **numbers measured over time**. They tell you what's happening at a high level.

```mermaid
flowchart LR
    API[API Server] -->|Every second| Metrics[(Metrics Store)]
    Metrics --> TP[Throughput: 1200 req/s]
    Metrics --> LAT[p99 latency: 230ms]
    Metrics --> ERR[Error rate: 0.02%]
    Metrics --> CPU[CPU usage: 72%]
```

**Common API metrics:**

| Metric | What it tells you |
|---|---|
| **Throughput** (RPS) | How many requests per second |
| **Latency** (p50, p95, p99) | How long requests take |
| **Error rate** | Percentage of requests returning 4xx/5xx |
| **Saturation** | How full is the queue / resource |

**Infrastructure metrics:**

| Metric | What it tells you |
|---|---|
| **CPU usage** | Is the server overloaded? |
| **Memory usage** | Is the app leaking memory? |
| **Disk I/O** | Is the database bottlenecked on disk? |
| **Network I/O** | How much data is flowing in/out? |
| **Disk space** | Will the server fill up soon? |

---

### Latency Percentiles Explained

Don't just measure **average latency** — it hides the pain felt by your slowest users.

```mermaid
flowchart LR
    Data["100 requests completed in: 10, 12, 15, 20, 25, 30, 40, 50, 100, 2000 ms"]
    Data --> Avg["Average: 230ms - misleading!"]
    Data --> P50["p50 (median): 25ms - half of users see this or faster"]
    Data --> P95["p95: 100ms - 95% of users see this or faster"]
    Data --> P99["p99: 2000ms - 1% of users wait 2 seconds"]
```

If you have 1 million requests per day, p99 means 10,000 users experience the worst latency. That matters.

---

### Pillar 2: Logs

Logs are **timestamped text records** of events that happened inside your system.

```mermaid
flowchart TD
    App[Application] -->|Writes| Logs["2024-01-15 10:23:45 INFO  User 1234 logged in
2024-01-15 10:23:46 ERROR DB connection timeout after 5000ms
2024-01-15 10:23:47 WARN  Retry attempt 1 for order-service
2024-01-15 10:23:50 INFO  Order 5678 created successfully"]
    
    Logs -->|Shipped to| ELK[(Elasticsearch / CloudWatch / Loki)]
    ELK --> Search[Search and filter logs]
    Search --> Debug[Debug specific errors]
```

**Log levels (most to least severe):**

```
FATAL  → System is crashing, cannot continue
ERROR  → Something failed (but the system might continue)
WARN   → Something unexpected, might cause issues
INFO   → Normal operational events ("user logged in")
DEBUG  → Detailed info for debugging (disabled in production)
```

**Structured logging** (JSON format) makes logs searchable:
```json
{
  "timestamp": "2024-01-15T10:23:46Z",
  "level": "ERROR",
  "message": "DB connection timeout",
  "service": "order-service",
  "user_id": 1234,
  "duration_ms": 5000,
  "trace_id": "abc-123-xyz"
}
```

---

### Pillar 3: Distributed Traces

When a single request flows through multiple services, how do you see the full journey?

A **trace** tracks one request as it travels through your system.

```mermaid
flowchart LR
    User -->|request| Gateway[API Gateway - 5ms]
    Gateway --> Auth[Auth Service - 10ms]
    Gateway --> Order[Order Service - 200ms]
    Order --> DB[(Database - 150ms)]
    Order --> Email[Email Service - 30ms]
    
    Total[Total: 245ms]
```

Without tracing, when a request takes 2 seconds, you don't know if it was slow at the gateway, auth, database, or email service.

With tracing:
```mermaid
gantt
    title Request trace (total: 245ms)
    dateFormat X
    axisFormat %Lms

    section API Gateway
    Routing & Auth check :0, 5

    section Auth Service
    Token validation :5, 15

    section Order Service
    Business logic :15, 65

    section Database
    Query execution :65, 215

    section Email Service
    Email send :215, 245
```

The database took 150ms → that's the bottleneck to fix.

---

## Health Checks

Every service should expose an endpoint that says "am I healthy?"

```mermaid
sequenceDiagram
    participant LB as Load Balancer / Orchestrator
    participant Service

    loop Every 10 seconds
        LB->>Service: GET /health
        Service-->>LB: 200 OK {"status": "healthy"}
    end

    Note over Service: Service becomes unhealthy
    LB->>Service: GET /health
    Service-->>LB: 500 {"status": "unhealthy", "reason": "DB unreachable"}
    LB->>LB: Remove this instance from rotation
```

**Types:**
- **Liveness probe**: Is the service alive? (restart if not)
- **Readiness probe**: Is the service ready to serve traffic? (stop sending it traffic if not)

---

## Alerting

Monitoring without alerting is like a smoke detector with no alarm.

```mermaid
flowchart TD
    Metrics --> Rules{Alert Rules}
    Rules -->|"Error rate > 1% for 5min"| Alert1[PagerDuty alert - page on-call engineer]
    Rules -->|"p99 latency > 2s"| Alert2[Slack message]
    Rules -->|"Disk usage > 90%"| Alert3[Slack message - plan ahead]
    Rules -->|"CPU > 95% sustained"| Alert4[Auto-scale + page engineer]
```

**Avoid alert fatigue** — if alerts fire too often for non-critical things, engineers start ignoring them.

**Golden rule:** Only alert on things that require **human action right now**.

---

## SLI, SLO, SLA

```mermaid
graph TD
    SLI["SLI - Service Level Indicator
    The actual metric you measure
    Example: 99.2% of requests completed in under 200ms"]
    
    SLO["SLO - Service Level Objective
    Your internal target
    Example: 99.5% of requests under 200ms"]
    
    SLA["SLA - Service Level Agreement
    Contract with the customer
    Example: 99.0% uptime or we issue refunds"]

    SLI -->|feeds into| SLO
    SLO -->|basis for| SLA
```

| Term | Audience | Consequence if missed |
|---|---|---|
| SLI | Engineering team | Dashboard turns red |
| SLO | Engineering team | Incident review, fix prioritized |
| SLA | Customers | Financial penalty / refund |

---

## Tools Overview

| Tool | Category | What it does |
|---|---|---|
| **Prometheus** | Metrics | Collects and stores metrics |
| **Grafana** | Visualization | Beautiful dashboards from metrics |
| **Datadog** | All-in-one | Metrics, logs, traces, alerts |
| **Elasticsearch + Kibana** | Logs | Search and visualize logs |
| **Jaeger / Zipkin** | Traces | Distributed request tracing |
| **PagerDuty** | Alerting | On-call alerting and escalation |
| **AWS CloudWatch** | All-in-one (AWS) | Managed monitoring on AWS |
| **Google Cloud Monitoring** | All-in-one (GCP) | Managed monitoring on GCP |
