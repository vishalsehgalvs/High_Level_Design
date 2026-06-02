# 07 — REST API Deep Dive

## REST in Plain English

REST is just a set of rules for building APIs over HTTP. It's not a protocol or a framework — it's a style.

The key idea: **every URL represents a "resource"** (a thing), and HTTP methods tell you what to do with it.

```mermaid
flowchart LR
    URL[URL = the thing] --> users[/users - the users collection]
    URL --> user[/users/42 - a specific user]
    URL --> posts[/users/42/posts - that user's posts]
    
    Method[HTTP Method = the action] --> GET[GET - read it]
    Method --> POST[POST - create it]
    Method --> PUT[PUT - replace it]
    Method --> PATCH[PATCH - update part of it]
    Method --> DELETE[DELETE - delete it]
```

---

## HTTP Methods

### GET — Read data (never changes anything)

```
GET /cars           → get all cars
GET /cars/5         → get car with id 5
GET /cars?brand=Toyota&year=2020  → filtered list
```

Rules for GET:
- **Safe** — reading doesn't change server state
- **Idempotent** — calling it 10 times gives the same result as calling it once
- **Cacheable** — responses can be cached

### POST — Create something new

```
POST /cars
Body: { "brand": "Toyota", "model": "Camry", "price": 25000 }

Response: 201 Created
Body: { "id": 43, "brand": "Toyota", "model": "Camry", "price": 25000 }
```

- **Not idempotent** — calling POST twice creates two cars
- Response code is `201 Created` (not `200 OK`)

### PUT — Replace an entire resource

```
PUT /cars/43
Body: { "brand": "Toyota", "model": "Corolla", "price": 20000 }
```

Replaces the whole object. If a field is missing from the body, it gets cleared.

### PATCH — Update only specific fields

```
PATCH /cars/43
Body: { "price": 22000 }
```

Only the `price` changes. Other fields stay the same.

### DELETE — Remove a resource

```
DELETE /cars/43
Response: 204 No Content
```

- `204` means "success, but no body to return"

---

## Request Anatomy

Every HTTP request has these parts:

```mermaid
graph TD
    Request[HTTP Request] --> Method[Method: GET / POST / PUT / DELETE]
    Request --> URL[URL: /api/v1/cars/42]
    Request --> Headers[Headers: Content-Type, Authorization, etc.]
    Request --> Body[Body: JSON data - only for POST / PUT / PATCH]
    Request --> QueryParams[Query Parameters: ?page=2&limit=10]
```

### Headers

Headers carry metadata about the request:

| Header | Purpose | Example |
|---|---|---|
| `Content-Type` | Format of the body | `application/json` |
| `Accept` | Format the client wants back | `application/json` |
| `Authorization` | Auth token | `Bearer eyJhbGciOiJIUzI1...` |
| `X-Request-ID` | Unique ID for tracing | `abc-123-xyz` |

### Query Parameters

Used for filtering, sorting, and paginating — never for identifying a specific resource.

```
GET /cars?brand=Toyota&year=2022&sort=price&order=asc&page=2&limit=20
```

| Param | Purpose |
|---|---|
| `brand=Toyota` | Filter by brand |
| `year=2022` | Filter by year |
| `sort=price` | Sort by field |
| `order=asc` | Sort direction |
| `page=2` | Page number |
| `limit=20` | Items per page |

### Request Body (JSON)

Used in POST / PUT / PATCH to send data to the server:

```json
{
    "brand": "Toyota",
    "model": "Camry",
    "year": 2022,
    "price": 28000,
    "features": ["sunroof", "backup camera"]
}
```

---

## Response Anatomy

```mermaid
graph TD
    Response[HTTP Response] --> StatusCode[Status Code: 200 / 201 / 400 / 404 / 500]
    Response --> Headers2[Headers: Content-Type, Cache-Control]
    Response --> Body2[Body: JSON data]
```

### Status Codes

```mermaid
graph LR
    2xx[2xx - Success] --> 200[200 OK - Request succeeded]
    2xx --> 201[201 Created - Resource created]
    2xx --> 204[204 No Content - Success, no body]
    
    3xx[3xx - Redirect] --> 301[301 Moved Permanently]
    3xx --> 302[302 Found - Temporary redirect]
    
    4xx[4xx - Client Error] --> 400[400 Bad Request - You sent garbage]
    4xx --> 401[401 Unauthorized - Not logged in]
    4xx --> 403[403 Forbidden - Logged in but not allowed]
    4xx --> 404[404 Not Found - Resource doesn't exist]
    4xx --> 409[409 Conflict - e.g. duplicate email]
    4xx --> 422[422 Unprocessable - Validation failed]
    4xx --> 429[429 Too Many Requests - Rate limited]
    
    5xx[5xx - Server Error] --> 500[500 Internal Server Error - Server crashed]
    5xx --> 502[502 Bad Gateway - Upstream server failed]
    5xx --> 503[503 Service Unavailable - Overloaded or down]
```

---

## RESTful URL Design Best Practices

### Use nouns, not verbs

```
❌  GET /getUser/42
❌  POST /createCar
❌  DELETE /deletePost/5

✅  GET /users/42
✅  POST /cars
✅  DELETE /posts/5
```

The verb comes from the HTTP method. The URL is just the resource.

### Use plural nouns for collections

```
✅  /users       (collection)
✅  /users/42    (specific item)
✅  /cars
✅  /cars/42/reviews   (nested - reviews of car 42)
```

### Use lowercase and hyphens

```
❌  /GetAllUsers
❌  /user_posts
✅  /users
✅  /user-posts
```

### Version your API

```
✅  /api/v1/users
✅  /api/v2/users
```

This lets you change the API without breaking existing clients.

---

## Full Request-Response Flow

```mermaid
sequenceDiagram
    participant Client as Mobile App
    participant LB as Load Balancer
    participant API as API Server
    participant Cache as Cache (Redis)
    participant DB as Database

    Client->>LB: GET /api/v1/cars/42
    LB->>API: Forwards request
    API->>Cache: Do you have car:42?
    Cache-->>API: Cache miss
    API->>DB: SELECT * FROM cars WHERE id = 42
    DB-->>API: Row data
    API->>Cache: Store car:42 for 5 minutes
    API-->>LB: 200 OK { "id": 42, "brand": "Toyota" }
    LB-->>Client: 200 OK { "id": 42, "brand": "Toyota" }
```

---

## Pagination Patterns

When a collection is huge, you don't return everything at once.

### Offset pagination (simplest)

```
GET /cars?page=3&limit=20
```

Returns items 41–60 (skip first 40, take 20).

### Cursor pagination (better for large datasets)

```
GET /cars?cursor=eyJpZCI6NDJ9&limit=20
```

The cursor is an encoded pointer to the last item seen. Better for infinite scroll.

```mermaid
flowchart LR
    Client -->|"GET /posts?cursor=abc123&limit=10"| Server
    Server -->|"Returns items + next_cursor=xyz789"| Client
    Client -->|"GET /posts?cursor=xyz789&limit=10"| Server
    Server -->|"Returns next 10 items"| Client
```
