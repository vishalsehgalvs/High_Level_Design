# 09 — NoSQL Databases: When SQL Isn't Enough

## Why NoSQL Exists

SQL databases are great. But they have weaknesses at scale:

1. **Scaling is hard**: SQL databases scale vertically (bigger machine) better than horizontally (more machines). Adding more servers to a SQL cluster is complex.
2. **Fixed schema**: Every row must match the table structure. Changing the schema on a huge live table is painful.
3. **Not all data is relational**: A JSON document, a social graph, or a time-series log doesn't fit nicely in tables.

NoSQL databases were built to solve these problems.

```mermaid
flowchart TD
    Problem[Problem with SQL at scale] --> Scale[Hard to split across multiple servers]
    Problem --> Schema[Schema changes are painful on huge tables]
    Problem --> Data[Some data doesn't fit in rows and columns]
    
    Scale --> NoSQL[NoSQL solves this]
    Schema --> NoSQL
    Data --> NoSQL
```

> **NoSQL ≠ "No SQL syntax"** — it means "Not Only SQL". Some NoSQL databases even support SQL-like queries.

---

## The 4 Types of NoSQL Databases

```mermaid
graph TD
    NoSQL[NoSQL Databases] --> KV[Key-Value Stores]
    NoSQL --> Doc[Document Databases]
    NoSQL --> Graph[Graph Databases]
    NoSQL --> Col[Columnar Databases]
```

---

## 1. Key-Value Stores

The simplest kind. Just like a Python dictionary: `key → value`.

```mermaid
flowchart LR
    K1[user:42] -->|value| V1["{'name': 'Alice', 'age': 28}"]
    K2[session:abc123] -->|value| V2["{'user_id': 42, 'expires': '2026-01-01'}"]
    K3[cart:user42] -->|value| V3["['item1', 'item2', 'item3']"]
```

**Used for:**
- Session storage
- Caching (most common use)
- Rate limiting counters
- Shopping carts

**Examples:** Redis, Memcached, DynamoDB (also supports other modes)

**Strengths:** Extremely fast (microsecond reads), simple to use
**Weaknesses:** Can't query by value — you must know the exact key

---

## 2. Document Databases

Stores data as **JSON-like documents**. Each document can have its own structure — no fixed schema.

```json
{
    "_id": "user_42",
    "name": "Alice",
    "email": "alice@example.com",
    "address": {
        "city": "Mumbai",
        "country": "India"
    },
    "tags": ["premium", "verified"],
    "last_login": "2026-06-01T10:00:00Z"
}
```

A different user document could have completely different fields. That's flexible schema.

```mermaid
flowchart LR
    App[Your App] -->|insert document| MongoDB[(MongoDB / Firestore)]
    App -->|query: find all users in Mumbai| MongoDB
    MongoDB -->|returns matching documents| App
```

**Used for:**
- User profiles
- Product catalogs
- Blog posts, comments
- Any data with variable structure

**Examples:** MongoDB, Firestore, CouchDB

**Strengths:** Flexible schema, easy to scale horizontally
**Weaknesses:** Joins are hard (you denormalize data instead)

---

## 3. Graph Databases

Designed specifically for data that's all about **relationships** — where the connections between things matter as much as the things themselves.

```mermaid
graph LR
    Alice((Alice)) -->|follows| Bob((Bob))
    Alice -->|follows| Carol((Carol))
    Bob -->|follows| Dave((Dave))
    Carol -->|friends with| Dave
    Dave -->|works at| Acme[Acme Corp]
    Alice -->|works at| Acme
```

Finding "friends of friends" in SQL requires expensive recursive joins. In a graph database, it's just traversing edges — fast and natural.

**Used for:**
- Social networks (who follows whom)
- Recommendation engines (users who bought X also bought Y)
- Fraud detection (identify suspicious transaction patterns)
- Knowledge graphs

**Examples:** Neo4j, Amazon Neptune

---

## 4. Columnar Databases (Wide-Column Stores)

Instead of organizing data by row, they organize it **by column**. Great for analytical queries on huge datasets.

```mermaid
flowchart TD
    RowDB[Row-oriented - stores one row at a time]
    ColDB[Column-oriented - stores one column at a time]
    
    RowDB -->|fast at| R1[Reading full records - SELECT *]
    ColDB -->|fast at| C1[Aggregating one field across millions of rows]
    C1 --> C2["SELECT AVG(price) FROM orders — reads only the price column"]
```

Also includes systems like **Cassandra** which use a wide-column model optimized for write-heavy distributed systems.

**Used for:**
- Analytics and data warehouses
- Time-series data (IoT sensor readings)
- High-write, distributed systems

**Examples:** Apache Cassandra, HBase, Google Bigtable, Amazon Redshift (analytics)

---

## SQL vs NoSQL

```mermaid
graph LR
    SQL --> sql1[Fixed schema]
    SQL --> sql2[Strong ACID guarantees]
    SQL --> sql3[Complex queries and joins]
    SQL --> sql4[Vertical scaling mainly]
    SQL --> sql5[Best for: Banking, ERP, any structured relational data]

    NoSQL --> nosql1[Flexible schema]
    NoSQL --> nosql2[Eventually consistent usually]
    NoSQL --> nosql3[Simple queries, denormalized data]
    NoSQL --> nosql4[Horizontal scaling designed-in]
    NoSQL --> nosql5[Best for: Social media, real-time apps, big data]
```

| | SQL | NoSQL |
|---|---|---|
| Schema | Fixed | Flexible |
| Consistency | Strong (ACID) | Usually eventual |
| Scaling | Vertical (harder horizontal) | Horizontal (designed for it) |
| Joins | Excellent | Difficult (avoid them) |
| Query power | Very high (full SQL) | Limited to simpler queries |
| Best for | Structured, relational data | Unstructured, high-volume data |

---

## The Real-World Answer: Use Both

Large systems typically use multiple databases for different needs:

```mermaid
flowchart TD
    App[Application] --> PG[(PostgreSQL - user accounts, orders, payments)]
    App --> Mongo[(MongoDB - product catalog, flexible attributes)]
    App --> Redis[(Redis - sessions, cache, rate limiting)]
    App --> Neo4j[(Neo4j - social graph, recommendations)]
    App --> Cassandra[(Cassandra - activity logs, time-series events)]
```

This is called **polyglot persistence** — using the right database for each job.
