# 08 — SQL Databases: Relational Data Done Right

## What is a Relational Database?

A relational database organizes data into **tables** (like spreadsheets) with rows and columns. The "relational" part means tables can be **related to each other** through keys.

```mermaid
erDiagram
    USERS {
        int id PK
        string name
        string email
        string password_hash
    }
    POSTS {
        int id PK
        int user_id FK
        string title
        string content
        datetime created_at
    }
    COMMENTS {
        int id PK
        int post_id FK
        int user_id FK
        string text
    }

    USERS ||--o{ POSTS : "writes"
    USERS ||--o{ COMMENTS : "writes"
    POSTS ||--o{ COMMENTS : "has"
```

---

## Tables, Rows, and Columns

| Term | What it is | Analogy |
|---|---|---|
| **Table** | A collection of related data | A spreadsheet tab |
| **Row** | One record / entry | One row in the spreadsheet |
| **Column** | A field / attribute | One column header |
| **Schema** | The structure definition | The column headers + their types |

**Example: users table**

| id | name | email | age |
|---|---|---|---|
| 1 | Alice | alice@example.com | 28 |
| 2 | Bob | bob@example.com | 35 |
| 3 | Carol | carol@example.com | 22 |

---

## Constraints

Constraints are rules that the database enforces on your data. They prevent bad data from ever entering the system.

```mermaid
graph TD
    Constraints --> NN[NOT NULL - field cannot be empty]
    Constraints --> UQ[UNIQUE - no two rows can have same value]
    Constraints --> PK[PRIMARY KEY - unique identifier for each row]
    Constraints --> FK[FOREIGN KEY - value must exist in another table]
    Constraints --> CHK[CHECK - custom rule like age > 0]
    Constraints --> DEF[DEFAULT - value if not provided]
```

```sql
CREATE TABLE users (
    id       INT PRIMARY KEY AUTO_INCREMENT,
    email    VARCHAR(255) NOT NULL UNIQUE,
    name     VARCHAR(100) NOT NULL,
    age      INT CHECK (age >= 0),
    role     VARCHAR(20) DEFAULT 'user'
);
```

---

## Primary Key and Foreign Key

### Primary Key (PK)
The unique identifier for a row. No two rows can have the same PK.

```
users table:
id (PK) | name  | email
1       | Alice | alice@example.com
2       | Bob   | bob@example.com
```

### Foreign Key (FK)
References the primary key of another table. Creates the "relation" in relational database.

```
posts table:
id (PK) | user_id (FK → users.id) | title
1       | 1                        | Alice's first post
2       | 1                        | Alice's second post
3       | 2                        | Bob's post
```

```mermaid
flowchart LR
    P[posts.user_id = 1] -->|references| U[users.id = 1]
```

If you try to insert a post with `user_id = 99` and there's no user with id 99, the database **rejects it**. That's referential integrity.

---

## Relationships

### One-to-One

One row in table A links to exactly one row in table B.

```mermaid
erDiagram
    USER ||--|| PROFILE : "has"
    USER { int id }
    PROFILE { int id, int user_id, string bio, string avatar_url }
```

Example: A user has one profile. A profile belongs to one user.

### One-to-Many

One row in table A links to many rows in table B.

```mermaid
erDiagram
    USER ||--o{ POST : "writes"
    USER { int id }
    POST { int id, int user_id, string content }
```

Example: One user can write many posts. Each post belongs to one user.

### Many-to-Many

Many rows in A link to many rows in B. Requires a **junction table**.

```mermaid
erDiagram
    STUDENT }o--o{ COURSE : "enrolls in"
    STUDENT { int id, string name }
    ENROLLMENT { int student_id FK, int course_id FK, date enrolled_at }
    COURSE { int id, string name }
```

Example: A student can take many courses. A course can have many students.

---

## Joins

Joins let you combine data from multiple tables into one result.

### INNER JOIN — only rows that match in BOTH tables

```sql
SELECT users.name, posts.title
FROM posts
INNER JOIN users ON posts.user_id = users.id;
```

```mermaid
flowchart LR
    Posts[(posts table)] -->|INNER JOIN on user_id = id| Users[(users table)]
    Users --> Result[Combined result]
```

| users.name | posts.title |
|---|---|
| Alice | Alice's first post |
| Alice | Alice's second post |
| Bob | Bob's post |

### LEFT JOIN — all rows from left table, matching from right (or NULL)

```sql
SELECT users.name, posts.title
FROM users
LEFT JOIN posts ON users.id = posts.user_id;
```

Returns all users, even if they have no posts (post fields will be NULL).

### Types of Joins at a Glance

```mermaid
graph LR
    J[Joins] --> IJ[INNER JOIN - only matching rows]
    J --> LJ[LEFT JOIN - all from left + matches from right]
    J --> RJ[RIGHT JOIN - all from right + matches from left]
    J --> FJ[FULL JOIN - all from both, NULLs where no match]
```

---

## When to Use SQL

```mermaid
graph TD
    SQL[Use SQL when...] --> ACID[You need ACID transactions - banking, orders]
    SQL --> Rel[Data has clear relationships]
    SQL --> Complex[Complex queries with joins and aggregations]
    SQL --> Consistent[Strong consistency is required]
    SQL --> Reports[Reporting and analytics]
```

**Popular SQL databases:**

| Database | Best for |
|---|---|
| **PostgreSQL** | Most use cases — powerful, open-source |
| **MySQL / MariaDB** | Web apps, widely supported |
| **SQLite** | Embedded, small apps, testing |
| **SQL Server** | Enterprise Microsoft stack |
| **Oracle** | Enterprise, banking |
