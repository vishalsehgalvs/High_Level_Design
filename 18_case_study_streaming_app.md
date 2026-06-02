# 18 — Case Study: Design a Video Streaming Platform

## The Problem

Design a system like **YouTube** or **Netflix** where:
- Users can **upload videos**
- Other users can **watch those videos** from anywhere in the world
- The system handles millions of concurrent viewers
- Videos load fast even on slow connections

This is a classic system design interview problem because it touches almost every concept covered in this course.

---

## Step 1: Clarify Requirements

### Functional Requirements
- Users can upload videos
- Users can watch videos
- Videos are available in multiple resolutions (360p, 720p, 1080p, 4K)
- Users can search for videos
- Videos have titles, descriptions, and tags
- Basic view count tracking

### Non-Functional Requirements
- **High availability** — 99.99% uptime (users expect streaming to always work)
- **Low latency** — videos should start playing within 2 seconds
- **Massive scale** — YouTube processes 500 hours of video per minute
- **Eventual consistency** — view counts can be slightly delayed; that's fine

### Scale Estimates
- 1 billion users
- 1 million video uploads per day
- 500 million video views per day
- Average video: 500MB (pre-compression)
- After transcoding, ~1GB per video across all resolutions

---

## Step 2: High-Level Architecture

```mermaid
flowchart TD
    subgraph "Upload Path"
        Uploader[User - Uploads video] --> UploadService[Upload Service]
        UploadService --> RawStorage[(Raw Video Storage - S3)]
        UploadService --> Queue[(Message Queue - Kafka)]
        Queue --> TranscodeWorkers[Transcoding Workers - Farm of servers]
        TranscodeWorkers --> CDN_Storage[(Processed Videos - S3 + CDN Origins)]
    end

    subgraph "Watch Path"
        Viewer[User - Watches video] --> CDN[CDN - Edge Servers worldwide]
        CDN -->|Cache miss| CDN_Storage
    end

    subgraph "Metadata + Search"
        UploadService --> MetadataDB[(Metadata DB - PostgreSQL)]
        TranscodeWorkers -->|Update status| MetadataDB
        MetadataDB --> SearchIndex[(Search Index - Elasticsearch)]
        Viewer --> SearchService[Search Service]
        SearchService --> SearchIndex
    end

    subgraph "View Counting"
        CDN --> ViewEvents[(View Events - Kafka)]
        ViewEvents --> CountWorker[Count Aggregation Worker]
        CountWorker --> ViewCountDB[(View Count DB - Redis / Cassandra)]
    end
```

---

## Step 3: Upload Pipeline (Deep Dive)

This is the most complex part of the system.

```mermaid
sequenceDiagram
    participant User
    participant UploadService as Upload Service
    participant S3_Raw as Raw S3 Bucket
    participant Kafka
    participant Transcoder as Transcoding Workers
    participant S3_CDN as Processed S3 / CDN

    User->>UploadService: Upload video (POST /videos)
    UploadService->>S3_Raw: Store raw video file
    UploadService->>Kafka: Publish "video.uploaded" event
    UploadService-->>User: 202 Accepted (processing in background)

    Kafka->>Transcoder: Pick up transcoding job
    Transcoder->>S3_Raw: Download raw video
    Transcoder->>Transcoder: Transcode to 360p, 720p, 1080p, 4K
    Transcoder->>S3_CDN: Upload all versions
    Transcoder->>Kafka: Publish "video.ready" event
    
    Note over User: User gets notification - video is live!
```

### Why asynchronous?
- Transcoding a 1-hour video at 4K can take **30+ minutes**
- You can't make the user wait with an open HTTP connection
- Use a message queue so the user gets an immediate response, and workers process in the background

---

## Step 4: Transcoding (Compute-Intensive Work)

Transcoding = converting a raw video file into multiple formats and resolutions.

```mermaid
flowchart LR
    Raw["Raw video
    1080p H.265 MOV
    10GB"] --> Transcode[Transcoding Farm]
    
    Transcode --> V360["360p H.264 MP4
    50MB"]
    Transcode --> V720["720p H.264 MP4
    200MB"]
    Transcode --> V1080["1080p H.264 MP4
    500MB"]
    Transcode --> V4K["4K H.265 MP4
    2GB"]
    Transcode --> Thumbnails["Thumbnails
    JPEGs at 0:00, 0:30, 1:00..."]
    Transcode --> Subtitles["Subtitle tracks"]
```

This is **compute-intensive** work (heavy CPU/GPU usage). You scale this by having a **fleet of workers** that pick jobs from the queue.

**Why multiple resolutions?**
- Mobile users on slow networks watch 360p
- TV viewers watch 4K
- The video player auto-selects based on network speed (adaptive bitrate streaming)

---

## Step 5: CDN Delivery (How Video Actually Reaches You Fast)

Without a CDN, every video request would travel from your browser to a central server. If that server is in Virginia and you're in Mumbai, that's 200ms of latency minimum — per segment.

```mermaid
flowchart LR
    subgraph "Without CDN"
        U1[User in Mumbai] -->|200ms round trip| Origin[Origin Server - Virginia]
    end

    subgraph "With CDN"
        U2[User in Mumbai] -->|5ms| Edge1[CDN Edge - Mumbai]
        U3[User in London] -->|5ms| Edge2[CDN Edge - London]
        Edge1 -->|Cache miss - first time only| Origin2[Origin - Virginia]
        Edge2 -->|Cache miss| Origin2
    end
```

**How video streaming with CDN works:**

Videos are broken into **small segments** (2-10 seconds each). The player requests segment by segment.

```mermaid
sequenceDiagram
    participant Player as Video Player
    participant CDN

    Player->>CDN: GET /video/abc/720p/segment_001.ts
    CDN-->>Player: ← 2-second video chunk
    Player->>CDN: GET /video/abc/720p/segment_002.ts
    CDN-->>Player: ← 2-second video chunk
    Note over Player: Playing segment 1 while fetching segment 2
```

This is called **HLS (HTTP Live Streaming)** or **DASH (Dynamic Adaptive Streaming)**.

---

## Step 6: Metadata Storage

Every video needs metadata: title, description, uploader, views, likes, tags, status.

```mermaid
erDiagram
    VIDEO {
        string video_id PK
        string title
        string description
        string uploader_id FK
        string status
        int duration_seconds
        timestamp created_at
    }
    USER {
        string user_id PK
        string username
        string email
    }
    VIDEO_FILE {
        string file_id PK
        string video_id FK
        string resolution
        string s3_url
        int file_size_bytes
    }
    
    USER ||--o{ VIDEO : uploads
    VIDEO ||--o{ VIDEO_FILE : has
```

- Use **PostgreSQL** for structured metadata (ACID, joins, reliable)
- Use **Elasticsearch** for search (full-text search on titles and descriptions)
- When a video is uploaded, write to PostgreSQL and index into Elasticsearch

---

## Step 7: Search

```mermaid
flowchart LR
    User[User searches: "cooking pasta"] --> SearchAPI[Search API]
    SearchAPI --> ES[(Elasticsearch)]
    ES -->|Ranked results| SearchAPI
    SearchAPI -->|Video IDs + metadata| User
    User -->|Click a video| CDN[Stream from CDN]
```

**Why Elasticsearch?**
- Handles typos ("pastaa" → "pasta")
- Relevance ranking (views, freshness, engagement)
- Fast full-text search across millions of records
- Not good at: transactional writes, complex relationships

---

## Step 8: View Count Tracking at Scale

If 1 million people watch a video simultaneously, you can't write to the database on every view — it would be overwhelmed.

```mermaid
flowchart TD
    Viewers["1M concurrent viewers"]
    Viewers -->|Each view generates event| Kafka[(Kafka - View Events)]
    Kafka --> Flink[Stream Processing - Apache Flink / Spark Streaming]
    Flink -->|Aggregate: video X has 50,000 views in last minute| Redis[(Redis - Fast view counts)]
    Flink -->|Batch insert every 60s| Cassandra[(Cassandra - Long-term view history)]
    
    API[Read API] --> Redis
```

- **Write path**: High volume events → Kafka → aggregated in memory → stored in Redis/Cassandra
- **Read path**: API reads from Redis (fast)
- Eventually consistent: view counts shown to users may be ~60 seconds delayed — that's acceptable

---

## Full Architecture Diagram

```mermaid
flowchart TD
    subgraph "Client"
        Web[Web Browser]
        Mobile[Mobile App]
        TV[Smart TV]
    end

    subgraph "Edge Layer"
        CDN[CDN - Akamai / Cloudflare / AWS CloudFront]
    end

    subgraph "API Layer"
        AG[API Gateway]
        AuthService[Auth Service]
        UploadService[Upload Service]
        SearchService[Search Service]
        VideoService[Video Metadata Service]
    end

    subgraph "Processing"
        Kafka[(Kafka)]
        Transcoder[Transcoding Worker Farm]
        ViewCounter[View Count Aggregator]
    end

    subgraph "Storage"
        S3_Raw[(S3 - Raw Videos)]
        S3_CDN[(S3 - Processed Videos - CDN Origin)]
        Postgres[(PostgreSQL - Metadata)]
        ES[(Elasticsearch - Search)]
        Redis[(Redis - View Counts Cache)]
        Cassandra[(Cassandra - View History)]
    end

    Web & Mobile & TV --> CDN
    CDN -->|API calls| AG
    CDN -->|Video segments cache miss| S3_CDN

    AG --> AuthService
    AG --> UploadService
    AG --> SearchService
    AG --> VideoService

    UploadService --> S3_Raw
    UploadService --> Kafka
    UploadService --> Postgres

    Kafka --> Transcoder
    Transcoder --> S3_CDN
    Transcoder --> Postgres

    Kafka --> ViewCounter
    ViewCounter --> Redis
    ViewCounter --> Cassandra

    SearchService --> ES
    VideoService --> Postgres
    VideoService --> Redis
```

---

## Key Trade-Offs Made

| Decision | Choice | Why |
|---|---|---|
| Upload acknowledgement | Async (202) | Can't block user for 30-min transcoding |
| View counts | Eventually consistent | Throughput over accuracy |
| Metadata storage | PostgreSQL | Structured, needs joins, ACID |
| Search | Elasticsearch | Full-text, ranking, fuzzy matching |
| Video delivery | CDN | Latency is king for video |
| High-scale write buffering | Kafka | Decouple producers from storage |
| Transcoding | Worker pool | CPU-intensive, scale horizontally |

---

## What Makes This System Design Hard

1. **Scale of writes** — millions of concurrent events. Can't write directly to SQL.
2. **Compute-intensive jobs** — transcoding needs separate horizontally scalable workers.
3. **Global low latency** — CDN is essential; there's no other way.
4. **Availability requirements** — people rage when Netflix goes down. 99.99% uptime.
5. **Data at rest size** — storing petabytes of video cheaply (S3 with tiered storage).
