# 🖥️ System Design — Complete Interview Preparation Notes

---

## 1. Client-Server Architecture

### What is it?
A distributed computing model where **clients** request resources/services and **servers** provide them.

### Key Components
- **Client**: Initiates requests (browser, mobile app, desktop app)
- **Server**: Processes requests and returns responses
- **Network**: The medium of communication (internet/intranet)

### How It Works

```
┌─────────────┐         Request          ┌─────────────┐
│             │ ───────────────────────► │             │
│   CLIENT    │                          │   SERVER    │
│  (Browser)  │ ◄─────────────────────── │  (Backend)  │
└─────────────┘         Response         └─────────────┘
```

### Types

| Type | Description | Example |
|------|-------------|---------|
| **2-Tier** | Client talks directly to DB | Desktop apps |
| **3-Tier** | Client → App Server → DB | Most web apps |
| **N-Tier** | Multiple intermediary layers | Enterprise apps |

### 3-Tier Architecture (Most Common)

```
┌──────────┐    ┌──────────────┐    ┌──────────────┐
│  CLIENT  │───►│  APP SERVER  │───►│   DATABASE   │
│(Presentation)│ │  (Business  │    │  (Data Tier) │
│          │◄───│    Logic)    │◄───│              │
└──────────┘    └──────────────┘    └──────────────┘
```

### Interview Tips
- Always clarify stateless vs stateful servers
- Stateless servers are easier to scale horizontally
- Clients can be **thin** (dumb terminal) or **thick** (heavy logic on client)

---

## 2. IP Address

### What is it?
A unique numerical label assigned to every device on a network for identification and location addressing.

### IPv4 vs IPv6

```
IPv4:  192.168.1.1          → 32-bit, ~4.3 billion addresses
IPv6:  2001:0db8::1         → 128-bit, ~340 undecillion addresses
```

### Types of IP Addresses

```
┌─────────────────────────────────────────────────┐
│                  IP ADDRESSES                   │
├───────────────────┬─────────────────────────────┤
│    PUBLIC IP      │        PRIVATE IP            │
│  (Internet-wide)  │   (Local network only)       │
│   e.g. 8.8.8.8   │   e.g. 192.168.x.x          │
│                   │       10.x.x.x               │
│                   │       172.16-31.x.x          │
└───────────────────┴─────────────────────────────┘
```

### Static vs Dynamic IP

| | Static | Dynamic |
|---|---|---|
| **Changes?** | Never | On reconnect (via DHCP) |
| **Use case** | Servers, DNS | Home users |
| **Cost** | Higher | Lower |

### Key Concepts
- **Loopback**: `127.0.0.1` — refers to the local machine itself
- **CIDR Notation**: `192.168.1.0/24` — defines a range of IPs (subnet)
- **NAT (Network Address Translation)**: Maps private IPs to a public IP

---

## 3. DNS (Domain Name System)

### What is it?
DNS is the **phonebook of the internet** — it translates human-readable domain names into IP addresses.

### DNS Resolution Flow

```
User types: www.google.com
                │
                ▼
        ┌───────────────┐
        │  DNS Resolver │  (Usually your ISP)
        │   (Recursive) │
        └───────┬───────┘
                │ Cache miss → asks Root
                ▼
        ┌───────────────┐
        │  Root Server  │  Knows who handles .com
        └───────┬───────┘
                ▼
        ┌───────────────┐
        │  TLD Server   │  Knows who handles google.com
        │   (.com)      │
        └───────┬───────┘
                ▼
        ┌───────────────┐
        │  Authoritative│  Returns: 142.250.80.46
        │  Name Server  │
        └───────────────┘
```

### DNS Record Types

| Record | Purpose | Example |
|--------|---------|---------|
| **A** | Domain → IPv4 | `google.com → 142.250.80.46` |
| **AAAA** | Domain → IPv6 | `google.com → 2607:f8b0::...` |
| **CNAME** | Alias to another domain | `www → google.com` |
| **MX** | Mail server | For email routing |
| **TXT** | Verification/metadata | SPF, DKIM records |
| **NS** | Name server | Points to authoritative server |

### TTL (Time to Live)
- DNS records have a TTL — how long they're cached
- Low TTL = faster propagation but more DNS queries
- High TTL = better performance but slower updates

### Interview Tips
- DNS is hierarchical and distributed for fault tolerance
- DNS caching happens at OS, browser, and ISP levels
- **DNS Propagation** can take up to 48 hours

---

## 4. Proxy / Reverse Proxy

### Forward Proxy
Sits between **client and internet**. Acts on behalf of the client.

```
┌────────┐     ┌─────────────────┐     ┌──────────┐
│ Client │────►│  Forward Proxy  │────►│ Internet │
└────────┘     └─────────────────┘     └──────────┘
```

**Use Cases**: Anonymity, content filtering, bypassing geo-restrictions

### Reverse Proxy
Sits between **internet and servers**. Acts on behalf of the server.

```
┌──────────┐     ┌─────────────────┐     ┌──────────────┐
│ Internet │────►│  Reverse Proxy  │────►│  Server 1    │
│ (Client) │     │   (e.g. Nginx)  │────►│  Server 2    │
└──────────┘     └─────────────────┘     │  Server 3    │
                                          └──────────────┘
```

**Use Cases**: Load balancing, SSL termination, caching, security, hiding server topology

### Key Differences

| | Forward Proxy | Reverse Proxy |
|---|---|---|
| **Acts for** | Client | Server |
| **Hides** | Client identity | Server identity |
| **Examples** | VPN, Squid | Nginx, HAProxy, Cloudflare |

### Interview Tips
- Nginx is both a web server and a reverse proxy
- Reverse proxy is often the entry point for microservices
- SSL termination at reverse proxy reduces server overhead

---

## 5. Latency

### What is it?
The **time delay** between sending a request and receiving a response. Measured in milliseconds (ms).

### Latency Numbers Every Engineer Should Know

```
┌─────────────────────────────────────────────────────┐
│  Operation                          │  Latency      │
├─────────────────────────────────────┼───────────────┤
│  L1 Cache reference                 │  ~0.5 ns      │
│  L2 Cache reference                 │  ~7 ns        │
│  RAM access                         │  ~100 ns      │
│  SSD random read                    │  ~100 µs      │
│  HDD seek                           │  ~10 ms       │
│  Network: Same datacenter           │  ~0.5 ms      │
│  Network: US to Europe              │  ~150 ms      │
│  Network: US to Australia           │  ~300 ms      │
└─────────────────────────────────────┴───────────────┘
```

### Types of Latency
- **Network Latency**: Time for data to travel over the network
- **Disk Latency**: Time to read/write data from storage
- **Application Latency**: Time for code to process the request
- **Tail Latency**: Latency at the 99th or 99.9th percentile (p99, p999)

### How to Reduce Latency

```
High Latency Problem → Solutions:
├── Use CDN (cache content near users)
├── Use caching (avoid repeated computation)
├── Optimize database queries + indexing
├── Use faster storage (SSD over HDD)
├── Minimize network round trips (batch requests)
└── Use async processing (don't wait for slow ops)
```

### Latency vs Throughput vs Bandwidth

| Term | Definition |
|------|-----------|
| **Latency** | Time for one request/response |
| **Throughput** | Requests handled per second |
| **Bandwidth** | Max data transfer rate (bits/sec) |

---

## 6. HTTP / HTTPS

### HTTP (HyperText Transfer Protocol)
Application-layer protocol for transmitting hypermedia. **Stateless** — each request is independent.

### HTTP Request Structure

```http
GET /api/users/1 HTTP/1.1
Host: api.example.com
Authorization: Bearer token123
Accept: application/json
```

### HTTP Response Structure

```http
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 123

{"id": 1, "name": "Alice"}
```

### HTTP Methods

| Method | Purpose | Idempotent? | Safe? |
|--------|---------|-------------|-------|
| **GET** | Retrieve data | ✅ | ✅ |
| **POST** | Create resource | ❌ | ❌ |
| **PUT** | Replace resource | ✅ | ❌ |
| **PATCH** | Partial update | ❌ | ❌ |
| **DELETE** | Remove resource | ✅ | ❌ |

### HTTP Status Codes

```
1xx → Informational  (100 Continue)
2xx → Success        (200 OK, 201 Created, 204 No Content)
3xx → Redirect       (301 Moved Permanently, 302 Found)
4xx → Client Error   (400 Bad Request, 401 Unauthorized, 404 Not Found, 429 Too Many Requests)
5xx → Server Error   (500 Internal Server Error, 503 Service Unavailable)
```

### HTTP Versions

| Version | Key Feature |
|---------|-------------|
| **HTTP/1.0** | One request per connection |
| **HTTP/1.1** | Keep-alive, pipelining |
| **HTTP/2** | Multiplexing, binary, header compression |
| **HTTP/3** | QUIC protocol (UDP-based), faster handshake |

### HTTPS
HTTP + **TLS (Transport Layer Security)** encryption.

```
Client                          Server
  │                               │
  │──── ClientHello ─────────────►│
  │◄─── ServerHello + Cert ───────│
  │──── Verify Cert, Send Key ───►│
  │◄─── Encrypted Session Key ────│
  │                               │
  │◄══════ Encrypted Data ════════│  (Secure Channel)
```

**Benefits of HTTPS**: Encryption, Data Integrity, Authentication

---

## 7. APIs

### What is it?
**Application Programming Interface** — a contract that defines how software components communicate.

### Types of APIs

```
┌─────────────────────────────────────────────────┐
│                   API TYPES                     │
├─────────────┬──────────────┬────────────────────┤
│    REST     │   GraphQL    │       gRPC          │
│  (HTTP)     │  (HTTP/WS)   │  (HTTP/2+Protobuf) │
│  Stateless  │  Flexible    │  High Performance   │
│  Widely     │  Queries     │  Microservices      │
│  Used       │              │                     │
└─────────────┴──────────────┴────────────────────┘
```

### API Authentication Methods

| Method | How It Works |
|--------|-------------|
| **API Key** | Static key in header/query param |
| **OAuth 2.0** | Token-based delegation |
| **JWT** | Signed token with claims |
| **Basic Auth** | Base64 encoded user:password |

### API Versioning Strategies

```
URL Path:    /api/v1/users
Header:      API-Version: 2
Query Param: /api/users?version=1
```

---

## 8. REST API

### What is it?
**Representational State Transfer** — an architectural style for designing networked applications using HTTP.

### 6 REST Constraints
1. **Stateless** — No client state stored on server
2. **Client-Server** — Separation of concerns
3. **Cacheable** — Responses must define cacheability
4. **Uniform Interface** — Consistent resource identification
5. **Layered System** — Client unaware of intermediaries
6. **Code on Demand** *(optional)* — Server can send executable code

### REST Resource Design

```
Resource: Users

GET    /users         → List all users
POST   /users         → Create new user
GET    /users/{id}    → Get specific user
PUT    /users/{id}    → Replace user
PATCH  /users/{id}    → Partial update
DELETE /users/{id}    → Delete user

Nested Resource:
GET    /users/{id}/posts  → Posts by user
```

### Good vs Bad REST Design

```
✅ Good:
GET /articles/123
DELETE /articles/123

❌ Bad:
GET /getArticle?id=123
POST /deleteArticle/123
```

### REST Response Conventions

```json
// Success
{
  "data": { "id": 1, "name": "Alice" },
  "status": "success"
}

// Error
{
  "error": {
    "code": 404,
    "message": "User not found"
  }
}
```

---

## 9. GraphQL

### What is it?
A **query language for APIs** and a runtime for executing queries. Lets clients request exactly the data they need.

### REST vs GraphQL

```
REST Problem (Over-fetching):
GET /users/1  →  Returns ALL fields even if you need only name

REST Problem (Under-fetching):
Need user + posts + comments = 3 separate API calls

GraphQL Solution — One request, exact data:
query {
  user(id: 1) {
    name
    posts {
      title
      comments {
        text
      }
    }
  }
}
```

### GraphQL Operations

```graphql
# Query (Read)
query GetUser($id: ID!) {
  user(id: $id) {
    name
    email
  }
}

# Mutation (Write)
mutation CreateUser($name: String!) {
  createUser(name: $name) {
    id
    name
  }
}

# Subscription (Real-time)
subscription OnNewMessage {
  messageSent {
    content
    sender
  }
}
```

### GraphQL Architecture

```
┌──────────┐      Single POST /graphql      ┌─────────────┐
│  Client  │ ─────────────────────────────► │   GraphQL   │
│          │ ◄───────────────────────────── │   Server    │
└──────────┘    Exact requested data        └──────┬──────┘
                                                   │
                                    ┌──────────────┼──────────────┐
                                    ▼              ▼              ▼
                                  User DB       Post DB      Comment DB
```

### When to Use GraphQL vs REST

| Use GraphQL When | Use REST When |
|-----------------|---------------|
| Complex, nested data | Simple CRUD operations |
| Multiple client types (mobile/web) | Public APIs (REST is universal) |
| Rapid product iteration | Caching is critical |
| Bandwidth constrained (mobile) | File uploads/downloads |

---

## 10. Databases

### What is it?
An organized collection of structured data, managed by a **DBMS (Database Management System)**.

### Database Categories

```
┌─────────────────────────────────────────────────────┐
│                    DATABASES                        │
├──────────────────────┬──────────────────────────────┤
│       SQL            │          NoSQL               │
│  (Relational)        │  (Non-Relational)            │
├──────────────────────┼──────────────────────────────┤
│  • PostgreSQL        │  • MongoDB (Document)        │
│  • MySQL             │  • Redis (Key-Value)         │
│  • SQLite            │  • Cassandra (Wide-Column)   │
│  • Oracle            │  • Neo4j (Graph)             │
│  • SQL Server        │  • DynamoDB (Key-Value)      │
└──────────────────────┴──────────────────────────────┘
```

### ACID Properties (SQL)

```
A — Atomicity:    All or nothing (transactions)
C — Consistency:  DB always in valid state
I — Isolation:    Transactions don't interfere
D — Durability:   Committed data persists (crash-safe)
```

---

## 11. SQL vs NoSQL

### SQL (Relational)

```sql
-- Table: Users
┌────┬───────┬──────────────────┐
│ id │ name  │     email        │
├────┼───────┼──────────────────┤
│  1 │ Alice │ alice@email.com  │
│  2 │ Bob   │ bob@email.com    │
└────┴───────┴──────────────────┘

-- Table: Orders
┌────┬─────────┬──────────┐
│ id │ user_id │  amount  │
├────┼─────────┼──────────┤
│  1 │    1    │  $99.00  │
└────┴─────────┴──────────┘
```

### NoSQL — Document Store (MongoDB)

```json
{
  "_id": "1",
  "name": "Alice",
  "email": "alice@email.com",
  "orders": [
    { "id": "1", "amount": 99.00 }
  ]
}
```

### Comparison Table

| Feature | SQL | NoSQL |
|---------|-----|-------|
| **Schema** | Fixed, predefined | Flexible, dynamic |
| **Scaling** | Vertical (mostly) | Horizontal |
| **Transactions** | Strong ACID | Eventual consistency |
| **Joins** | Yes | Rare/No |
| **Query Language** | SQL | API/custom |
| **Best For** | Complex relations, reporting | High volume, unstructured |

### NoSQL Types

```
Key-Value:    Redis, DynamoDB     → Caching, sessions
Document:     MongoDB, Firestore  → User profiles, catalogs
Wide-Column:  Cassandra, HBase    → Time-series, IoT data
Graph:        Neo4j               → Social networks, recommendations
```

### Interview Tips
- SQL for **financial systems**, **reporting**, **complex queries**
- NoSQL for **high write throughput**, **unstructured data**, **global scale**
- Not mutually exclusive — use **polyglot persistence**

---

## 12. Vertical Scaling

### What is it?
**Scale Up** — Adding more resources (CPU, RAM, Disk) to an existing machine.

```
Before Scaling:          After Vertical Scaling:
┌───────────┐            ┌──────────────────────┐
│ Server    │            │ BIGGER Server        │
│ 4 CPU     │   ──────►  │ 32 CPU               │
│ 16 GB RAM │            │ 256 GB RAM           │
│ 500 GB    │            │ 4 TB SSD             │
└───────────┘            └──────────────────────┘
```

### Pros & Cons

| Pros | Cons |
|------|------|
| Simple — no code changes | Hardware limits exist |
| No distributed complexity | Single point of failure |
| Works for most small systems | Expensive at high end |
| Low latency (no network hops) | Downtime during upgrades |

### When to Use
- Databases (easier to scale vertically first)
- Applications with strong consistency requirements
- When horizontal scaling complexity is not justified

---

## 13. Horizontal Scaling

### What is it?
**Scale Out** — Adding more machines to distribute the load.

```
Before:                    After Horizontal Scaling:
┌───────────┐              ┌────────────┐
│ Server 1  │              │  Server 1  │
│ (Overloaded)│   ───────► │  Server 2  │  ← Load Balancer
└───────────┘              │  Server 3  │     distributes
                           └────────────┘      traffic
```

### Pros & Cons

| Pros | Cons |
|------|------|
| Theoretically unlimited scale | Distributed system complexity |
| No single point of failure | Data consistency challenges |
| Cost-effective (commodity hardware) | Need stateless servers |
| Rolling updates possible | Network overhead |

### Stateless Design (Required for Horizontal Scaling)

```
❌ Stateful (Can't scale out):
Server stores session in memory → User must hit same server

✅ Stateless (Can scale out):
Session stored in Redis/DB → Any server can handle request
```

---

## 14. Load Balancers

### What is it?
Distributes incoming network traffic across multiple servers to ensure no single server is overwhelmed.

```
                    ┌─────────────────┐
        Requests    │                 │──►  Server 1
───────────────────►│  Load Balancer  │──►  Server 2
                    │                 │──►  Server 3
                    └─────────────────┘
                    Health checks all servers
```

### Load Balancing Algorithms

| Algorithm | How It Works | Best For |
|-----------|-------------|----------|
| **Round Robin** | Rotate through servers in order | Equal capacity servers |
| **Weighted Round Robin** | More requests to stronger servers | Mixed capacity |
| **Least Connections** | Send to server with fewest active connections | Long-lived connections |
| **IP Hash** | Hash client IP → same server always | Session stickiness |
| **Random** | Pick random server | Simple, uniform loads |

### Layer 4 vs Layer 7 Load Balancing

```
Layer 4 (Transport):
  - Routes based on IP + TCP/UDP port
  - Fast, simple
  - Can't inspect HTTP content

Layer 7 (Application):
  - Routes based on URL, headers, cookies
  - Can route /api/* → API servers
  - Can route /static/* → CDN
  - More flexible, slightly slower
```

### Health Checks

```
Load Balancer pings each server every N seconds:
  ✅ Server responds → Keep in rotation
  ❌ Server fails → Remove from rotation, alert
  ✅ Server recovers → Add back to rotation
```

### Types
- **Hardware LB**: F5, Citrix (expensive, fast)
- **Software LB**: Nginx, HAProxy (flexible, cheap)
- **Cloud LB**: AWS ALB/NLB, GCP Load Balancer

---

## 15. Indexing

### What is it?
A data structure that improves the **speed of data retrieval** at the cost of additional storage and slower writes.

### Without vs With Index

```sql
-- Without Index (Full Table Scan):
SELECT * FROM users WHERE email = 'alice@email.com'
-- → Scans ALL rows: O(n)

-- With Index on email column:
-- → Uses B-tree lookup: O(log n)
```

### How a B-Tree Index Works

```
              [M]
            /     \
         [D,G]   [P,T]
        /  |  \   |   \
      [A] [E] [H] [N] [U]

Binary search through tree → find row's disk location → fetch data
```

### Types of Indexes

| Type | Description | Use Case |
|------|-------------|----------|
| **B-Tree** | Balanced tree, default | Range queries, equality |
| **Hash** | Hash map | Exact match only |
| **Composite** | Multiple columns | Multi-column WHERE clauses |
| **Full-Text** | Tokenized text search | Search engines |
| **Partial** | Index subset of rows | WHERE is_active = true |
| **Clustered** | Data stored in index order | Primary key |
| **Non-Clustered** | Separate from data | Secondary lookups |

### Index Trade-offs

```
✅ Speeds up:  SELECT, WHERE, JOIN, ORDER BY
❌ Slows down: INSERT, UPDATE, DELETE (must update index)
❌ Uses:       Additional disk space
```

### Interview Tips
- **Cardinality** matters — index on gender (2 values) is useless
- **Composite index** order matters: `(a, b, c)` can serve `(a)`, `(a,b)`, `(a,b,c)` queries
- Avoid over-indexing — every index slows down writes

---

## 16. Replication

### What is it?
Copying data across multiple nodes/servers to ensure **availability** and **fault tolerance**.

### Primary-Replica (Master-Slave)

```
        ┌──────────────┐
        │   PRIMARY    │  ← All writes go here
        │   (Master)   │
        └──────┬───────┘
               │  Replication (async/sync)
       ┌───────┼───────┐
       ▼       ▼       ▼
  ┌────────┐ ┌────────┐ ┌────────┐
  │Replica1│ │Replica2│ │Replica3│  ← Reads distributed here
  └────────┘ └────────┘ └────────┘
```

### Synchronous vs Asynchronous Replication

| | Synchronous | Asynchronous |
|--|---|---|
| **Write confirmed when** | All replicas written | Primary written only |
| **Data loss risk** | None | Some (lag window) |
| **Write speed** | Slower | Faster |
| **Consistency** | Strong | Eventual |

### Primary-Primary (Multi-Master)

```
┌──────────┐  ←──────────►  ┌──────────┐
│ Primary 1│  (Bidirectional │ Primary 2│
│ (Writes) │   Replication) │ (Writes) │
└──────────┘                └──────────┘
```
- Both nodes accept writes
- Conflict resolution needed
- Higher availability

### Replication Lag Problem

```
Time 0:  Write to Primary → user_balance = $100
Time 1:  Replica not yet updated → user_balance = $50
Time 2:  Read hits Replica → Returns $50 ❌ (stale read)
Time 3:  Replica synced → Returns $100 ✅
```

**Solutions**: Read-your-writes consistency, route writes+reads to primary, use sync replication

---

## 17. Sharding

### What is it?
**Horizontal partitioning** of data across multiple database instances. Each shard contains a subset of the data.

```
Without Sharding:                With Sharding:
┌───────────────────┐            ┌──────────┐ ┌──────────┐ ┌──────────┐
│   Single DB       │            │ Shard 1  │ │ Shard 2  │ │ Shard 3  │
│   (All 100M rows) │   ──────►  │ Users    │ │ Users    │ │ Users    │
│   (Overloaded)    │            │ A-J      │ │ K-R      │ │ S-Z      │
└───────────────────┘            └──────────┘ └──────────┘ └──────────┘
```

### Sharding Strategies

```
1. Range-Based Sharding:
   User IDs 1-1M   → Shard 1
   User IDs 1M-2M  → Shard 2
   ⚠️ Risk: Hot shards if traffic uneven

2. Hash-Based Sharding:
   shard = hash(user_id) % num_shards
   ✅ Even distribution
   ⚠️ Risk: Rebalancing pain when adding shards

3. Geographic Sharding:
   US users → US Shard
   EU users → EU Shard
   ✅ Data locality, compliance
```

### Challenges of Sharding

```
❌ Cross-shard JOINs       → Expensive/complex
❌ Cross-shard transactions → Hard to maintain ACID
❌ Rebalancing             → Adding shard requires data migration
❌ Schema changes           → Must update all shards
```

### Interview Tips
- Start with vertical scaling, then replication, then sharding
- **Consistent hashing** minimizes data movement when adding/removing shards
- Application-level vs database-level sharding

---

## 18. Vertical Partitioning

### What is it?
Splitting a table by **columns** rather than rows. Different columns stored in different tables or databases.

```
Before (One fat table):
┌──────────────────────────────────────────────────┐
│ users: id │ name │ email │ bio │ profile_pic │... │
└──────────────────────────────────────────────────┘

After Vertical Partitioning:
┌───────────────────────┐   ┌──────────────────────┐
│ users_core            │   │ users_profile         │
│ id │ name │ email     │   │ id │ bio │ profile_pic│
└───────────────────────┘   └──────────────────────┘
     (Accessed frequently)        (Accessed rarely)
```

### Why Use It?
- Frequently accessed columns load faster (smaller row size)
- Sensitive columns (e.g., passwords) can be in separate table with stricter access
- Different storage engines for different access patterns
- Reduces I/O — queries only load needed columns

### Column Store Databases

```
Row Store (Traditional):    Column Store (e.g. Redshift):
Row 1: [1, Alice, 25]       col_id:   [1, 2, 3]
Row 2: [2, Bob, 30]    vs.  col_name: [Alice, Bob, ...]
Row 3: [3, Carol, 28]       col_age:  [25, 30, 28]

Best for OLTP                Best for OLAP/Analytics
(many small transactions)    (aggregations over columns)
```

---

## 19. Caching

### What is it?
Storing frequently accessed data in fast storage (usually in-memory) to reduce latency and database load.

### Caching Layers

```
Request Flow:
Browser → CDN Cache → Load Balancer → App Server
                                         │
                                    Redis Cache
                                         │ (miss)
                                      Database
```

### Cache Strategies

#### 1. Cache-Aside (Lazy Loading)

```
App checks cache →
  HIT:  Return cached data
  MISS: Fetch from DB → Store in cache → Return data
```

#### 2. Write-Through

```
Write to DB → Simultaneously write to cache
✅ Cache always fresh
❌ Every write hits cache (even cold data)
```

#### 3. Write-Behind (Write-Back)

```
Write to cache → Asynchronously write to DB later
✅ Fast writes
❌ Data loss risk if cache fails before DB write
```

#### 4. Read-Through

```
Cache sits in front of DB
App always reads from cache
Cache handles DB reads on miss automatically
```

### Cache Eviction Policies

| Policy | Description |
|--------|-------------|
| **LRU** (Least Recently Used) | Evict least recently accessed |
| **LFU** (Least Frequently Used) | Evict least accessed overall |
| **FIFO** | Evict oldest entry |
| **TTL** | Evict after time expires |

### Cache Problems

```
Cache Stampede:   Cache expires → 1000 requests hit DB simultaneously
Solution:         Add jitter to TTL, use locks, probabilistic refresh

Cache Penetration: Query for non-existent data → always misses
Solution:          Cache null results, bloom filters

Cache Avalanche:  Many caches expire at same time → DB overwhelmed
Solution:          Stagger TTLs, circuit breakers
```

### Interview Tips
- Redis vs Memcached: Redis has persistence, data structures, pub/sub
- Cache invalidation is one of the hardest problems in CS
- Always think about consistency between cache and DB

---

## 20. Denormalization

### What is it?
Intentionally introducing **redundancy** into a database to improve read performance at the cost of write performance and storage.

### Normalization vs Denormalization

```sql
-- Normalized (3NF):
-- orders table: id, user_id, product_id
-- users  table: id, name, email
-- Query needs: JOIN orders + users + products

-- Denormalized:
-- orders table: id, user_name, user_email, product_name, product_price
-- Query needs: Single table scan — no JOINs!
```

### When to Denormalize
- Read-heavy workloads (reporting, analytics)
- JOINs are too slow at scale
- When data rarely changes (avoids update anomalies)

### Denormalization Techniques

```
1. Duplicate Columns      → Store user_name directly in orders
2. Pre-computed Values    → Store order_total instead of computing
3. Materialized Views     → Pre-computed query results stored as table
4. Embedding (NoSQL)      → Store related docs inside parent document
```

---

## 21. CAP Theorem

### What is it?
In a distributed system, you can only guarantee **2 of 3** properties simultaneously:

```
              Consistency
                  /\
                 /  \
                /    \
               /  ??? \
              /________\
     Availability      Partition
                       Tolerance
```

### The Three Properties

| Property | Definition |
|----------|-----------|
| **Consistency (C)** | Every read returns the most recent write |
| **Availability (A)** | Every request gets a response (not guaranteed to be latest) |
| **Partition Tolerance (P)** | System continues despite network partitions |

### CAP in Practice

> **Network partitions are inevitable** in distributed systems, so you always choose P, then decide between C and A.

```
CP Systems (Consistent + Partition Tolerant):
  → Returns error if can't guarantee latest data
  → Examples: HBase, MongoDB (with strong consistency), Zookeeper
  → Use when: Banking, inventory (wrong data = disaster)

AP Systems (Available + Partition Tolerant):
  → Returns possibly stale data, but always responds
  → Examples: Cassandra, DynamoDB, CouchDB
  → Use when: Social feeds, DNS, shopping carts
```

### PACELC Extension

```
If Partition:     Choose between Availability or Consistency
Else (no partition): Choose between Latency or Consistency

PACELC captures the latency-consistency trade-off during normal operation
```

---

## 22. Blob Storage

### What is it?
**Binary Large Object** storage — designed for storing unstructured data like images, videos, audio, and documents.

```
┌──────────────────────────────────────────────────────┐
│                  BLOB STORAGE                        │
├───────────────┬──────────────────────────────────────┤
│  Object Store │  Files stored as objects with        │
│               │  unique keys (not file paths)        │
├───────────────┼──────────────────────────────────────┤
│  Flat         │  No directory hierarchy              │
│  Structure    │  (simulated by key naming)           │
├───────────────┼──────────────────────────────────────┤
│  HTTP Access  │  Access via URLs                     │
└───────────────┴──────────────────────────────────────┘
```

### Architecture

```
Client Upload Flow:
Client ──────────────────────────────────────► Blob Storage
         PUT /bucket/user-123/avatar.jpg          (S3/GCS/Azure)

Client Download Flow:
Client ◄────────────────────────────────────── Blob Storage
         GET https://bucket.s3.amazonaws.com/...
```

### Examples

| Provider | Service |
|----------|---------|
| AWS | S3 (Simple Storage Service) |
| Google Cloud | Cloud Storage (GCS) |
| Azure | Azure Blob Storage |
| Self-hosted | MinIO |

### Key Features
- **Virtually unlimited** storage
- **Durability**: AWS S3 offers 11 9s (99.999999999%)
- **Access Control**: Public/private, signed URLs
- **Lifecycle Policies**: Auto-archive to cold storage
- **Versioning**: Keep history of objects

### Signed URLs

```
Server generates temporary URL:
GET https://s3.amazonaws.com/bucket/file?
    X-Amz-Signature=abc&
    X-Amz-Expires=3600    ← expires in 1 hour

Client downloads directly from S3 (server not involved)
✅ Reduces server bandwidth costs
```

---

## 23. CDN (Content Delivery Network)

### What is it?
A globally distributed network of servers that **caches content close to users** to reduce latency.

```
Without CDN:
User in India ──────────────────────────────► Origin Server (US)
              ~~~ 300ms round trip ~~~

With CDN:
User in India ──────► CDN Edge (Mumbai) ──► Cached Response
              ~~~ 10ms ~~~
```

### CDN Architecture

```
                        ┌──────────────────┐
                        │  Origin Server   │
                        │     (US)         │
                        └────────┬─────────┘
                                 │
         ┌───────────────────────┼───────────────────────┐
         │                       │                       │
         ▼                       ▼                       ▼
 ┌──────────────┐       ┌──────────────┐       ┌──────────────┐
 │  Edge Node   │       │  Edge Node   │       │  Edge Node   │
 │  New York    │       │  London      │       │  Singapore   │
 └──────────────┘       └──────────────┘       └──────────────┘
         │                       │                       │
    US Users                EU Users              Asia Users
```

### How CDN Works

```
1. User requests: https://cdn.example.com/image.jpg
2. DNS resolves to nearest CDN edge node
3. Edge node has cache HIT → serves immediately
4. Cache MISS → Edge fetches from origin, caches, serves
5. Subsequent requests → served from edge cache
```

### What CDNs Cache

```
✅ Good for CDN:
   - Static assets (CSS, JS, images, fonts)
   - Video streaming
   - Software downloads
   - API responses (with proper cache headers)

❌ Not for CDN:
   - User-specific data
   - Real-time data
   - POST/PUT/DELETE requests
```

### CDN Providers
- Cloudflare, AWS CloudFront, Akamai, Fastly, Azure CDN

---

## 24. WebSockets

### What is it?
A **full-duplex**, persistent communication channel over a single TCP connection — enabling real-time bidirectional communication.

### HTTP Polling vs WebSockets

```
HTTP Polling (Inefficient):
Client: "Any new messages?" → Server: "No"  (every 2 seconds)
Client: "Any new messages?" → Server: "No"
Client: "Any new messages?" → Server: "Yes! Here's a message"

WebSocket (Efficient):
Client ──── Upgrade to WebSocket ────► Server
              (Single TCP connection)
Server ──── Push message anytime ────► Client  ✅
Client ──── Send message anytime ────► Server  ✅
```

### WebSocket Handshake

```http
-- 1. Client sends HTTP Upgrade request:
GET /ws HTTP/1.1
Upgrade: websocket
Connection: Upgrade

-- 2. Server responds:
HTTP/1.1 101 Switching Protocols
Upgrade: websocket

-- 3. Full-duplex channel established
--    ws:// or wss:// (secure)
```

### Use Cases

```
✅ Real-time chat
✅ Live sports scores
✅ Collaborative editing (Google Docs)
✅ Stock price feeds
✅ Multiplayer gaming
✅ Live notifications
```

### WebSocket vs SSE vs Long Polling

| | WebSocket | SSE | Long Polling |
|--|---|---|---|
| **Direction** | Bidirectional | Server→Client only | Client pulls |
| **Protocol** | WS | HTTP | HTTP |
| **Complexity** | Medium | Low | Low |
| **Use Case** | Chat, gaming | Notifications, feeds | Simple updates |

---

## 25. Webhooks

### What is it?
**Webhooks** are HTTP callbacks — when an event occurs, the server sends an HTTP POST to a pre-configured URL (your endpoint).

### Polling vs Webhooks

```
API Polling:
Your Server: "Did anything happen?" → API: "No"  (every minute)
Your Server: "Did anything happen?" → API: "No"
Your Server: "Did anything happen?" → API: "Yes! Payment succeeded"
Problem: Wasted requests, delayed response

Webhook (Event-driven):
Event occurs at Stripe (payment succeeded)
Stripe → POST https://yourapp.com/webhooks/stripe
           { "event": "payment.succeeded", "amount": 9900 }
✅ Instant notification, no wasted polling
```

### Webhook Flow

```
┌──────────────┐  Event  ┌─────────────────┐  HTTP POST  ┌──────────────┐
│   Provider   │────────►│  Webhook Engine  │────────────►│  Your Server │
│  (Stripe,    │         │                  │             │  /webhook    │
│   GitHub)    │         │                  │◄────────────│  endpoint    │
└──────────────┘         └─────────────────┘   200 OK    └──────────────┘
```

### Webhook Best Practices

```
1. Verify signatures:
   Stripe sends: X-Stripe-Signature header
   You compute HMAC and compare → prevents spoofing

2. Respond quickly (< 5s):
   Accept webhook → 200 OK immediately
   Process asynchronously in background queue

3. Idempotency:
   Same webhook may be delivered multiple times
   Use event ID to deduplicate

4. Retry handling:
   Provider retries on failure
   Your endpoint must handle duplicates gracefully
```

---

## 26. Microservices

### What is it?
An architectural pattern where an application is built as a collection of **small, independent services** that communicate over APIs.

### Monolith vs Microservices

```
Monolith:
┌───────────────────────────────────────────────┐
│         Single Application                    │
│  [User Module] [Order Module] [Payment Module]│
│  [Inventory]   [Notification] [Auth Module]   │
└───────────────────────────────────────────────┘
One codebase, one deployment, one database

Microservices:
┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│  User    │  │  Order   │  │ Payment  │  │  Notif.  │
│ Service  │  │ Service  │  │ Service  │  │ Service  │
│ (DB)     │  │ (DB)     │  │ (DB)     │  │ (DB)     │
└──────────┘  └──────────┘  └──────────┘  └──────────┘
Each independently deployable, scalable, developed
```

### Key Characteristics

```
✅ Each service has its own database (loose coupling)
✅ Services communicate via APIs or message queues
✅ Independent deployment per service
✅ Different teams own different services
✅ Can use different tech stacks per service
```

### Inter-Service Communication

```
Synchronous:           Asynchronous:
Service A              Service A
    │                      │
    │ HTTP/gRPC             │ Publish event
    ▼                      ▼
Service B           Message Queue (Kafka/RabbitMQ)
                           │
                           ▼
                       Service B (consumes)
```

### Microservices Challenges

| Challenge | Solution |
|-----------|---------|
| Service discovery | Consul, Kubernetes DNS |
| Distributed tracing | Jaeger, Zipkin |
| Data consistency | Saga pattern, eventual consistency |
| Network failures | Circuit breaker (Hystrix) |
| Config management | Vault, ConfigMaps |
| Testing complexity | Contract testing, integration tests |

### When NOT to Use Microservices
- Small teams (≤5 engineers)
- Early-stage products (premature optimization)
- When you're unsure of service boundaries

---

## 27. Message Queues

### What is it?
An **asynchronous communication mechanism** where producers send messages to a queue, and consumers process them independently.

### Architecture

```
┌──────────┐    Publish    ┌───────────────┐    Consume    ┌──────────┐
│ Producer │──────────────►│ Message Queue │──────────────►│ Consumer │
│ (App A)  │               │   (Buffer)    │               │ (App B)  │
└──────────┘               └───────────────┘               └──────────┘
                                  │
                            Messages stored
                            until consumed
```

### Why Use Message Queues?

```
Problem 1: Slow downstream service
Order Service → Email Service (slow!)
Without Queue: Order Service waits 3 seconds
With Queue:    Order Service publishes → continues → Email consumes later ✅

Problem 2: Traffic spikes
10,000 requests/sec hit your API
Without Queue: DB overwhelmed, crashes
With Queue:    10,000 msgs/sec into queue, DB processes at 1,000/sec ✅

Problem 3: Service unavailability
Without Queue: Request fails if downstream is down
With Queue:    Message waits in queue, processed when service recovers ✅
```

### Message Queue Types

```
Point-to-Point (Queue):          Publish-Subscribe (Topic):
Producer ──► Queue ──► Consumer  Producer ──► Topic ──► Consumer A
                                                    ──► Consumer B

One message, one consumer        One message, many consumers
Example: RabbitMQ queues         Example: Kafka topics
```

### Popular Message Queues

| System | Best For |
|--------|---------|
| **Kafka** | High throughput, event streaming, log aggregation |
| **RabbitMQ** | Complex routing, traditional message brokering |
| **AWS SQS** | Simple managed queuing, AWS ecosystem |
| **Redis Pub/Sub** | Simple pub/sub, ephemeral messages |

### Key Concepts

```
Dead Letter Queue (DLQ):
  Failed messages → DLQ → Manual investigation
  Prevents poison messages from blocking queue

Message Acknowledgment:
  Consumer processes message → sends ACK → queue deletes message
  No ACK (crash) → message redelivered to another consumer

Delivery Guarantees:
  At-Least-Once:  May redeliver (design for idempotency)
  At-Most-Once:   May lose messages (fire and forget)
  Exactly-Once:   Hardest, most expensive guarantee
```

---

## 28. Rate Limiting

### What is it?
Controlling the rate of requests a client can make to an API or service within a given time window.

### Why Rate Limit?

```
Without Rate Limiting:
- One client sends 1M requests/min → Server crashes
- DDoS attacks overwhelm the system
- Resource starvation for other users
- Runaway scripts exhaust API quotas
```

### Rate Limiting Algorithms

#### 1. Token Bucket

```
Bucket holds N tokens
Each request consumes 1 token
Tokens refill at rate R per second

[■■■■■] ← 5 tokens
Request → [■■■■ ] ← consume 1
Request → [■■■  ]
Request → [■■   ]
...Refill...
[■■■■■] ← full again

✅ Allows bursting up to bucket size
```

#### 2. Leaky Bucket

```
Requests enter at any rate → Queue (bucket) → Processed at fixed rate
Like a bucket with a hole — drains at constant rate
✅ Smooth, constant output rate
❌ Bursty traffic gets queued/dropped
```

#### 3. Fixed Window Counter

```
Window: [0s ─────────── 60s]
Count requests in window → if > limit → reject

Problem: Burst at boundary:
[....│■■■■■■■■■] [■■■■■■■■■│....]
 Prev window      Next window
60 req at 59s + 60 req at 61s = 120 in ~2s ❌
```

#### 4. Sliding Window Log

```
Keep timestamp of each request
Count requests in last N seconds
Most accurate, but high memory usage
```

### Rate Limit Response

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 60
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1620000000
```

### Rate Limiting in Distributed Systems

```
Problem: 10 app servers, rate limit = 100 req/min
Each server allows 100 → actually 1000 req/min allowed ❌

Solution: Centralized rate limiting with Redis
All servers check/update same Redis counter ✅
```

---

## 29. API Gateways

### What is it?
A single entry point for all client requests that **routes, transforms, and manages** API calls to backend services.

### API Gateway Architecture

```
                    ┌─────────────────────────────┐
                    │         API GATEWAY          │
                    │                             │
Clients ──────────► │  • Authentication           │──► User Service
(Web/Mobile/3rd)    │  • Rate Limiting            │──► Order Service
                    │  • Request Routing          │──► Payment Service
                    │  • Load Balancing           │──► Notification Service
                    │  • SSL Termination          │
                    │  • Request/Response Transform│
                    │  • Logging & Monitoring      │
                    └─────────────────────────────┘
```

### Key Functions

| Function | Description |
|----------|-------------|
| **Routing** | Route `/api/users` → User Service |
| **Auth** | Validate JWT/API keys before reaching services |
| **Rate Limiting** | Enforce per-client limits |
| **Caching** | Cache common responses |
| **SSL Termination** | Handle HTTPS at gateway |
| **Request Aggregation** | Combine multiple service calls |
| **Protocol Translation** | REST → gRPC |
| **Logging** | Centralized request logging |

### Popular API Gateways
- **AWS API Gateway** — Managed, serverless
- **Kong** — Open source, extensible
- **Nginx** — Lightweight, high performance
- **Apigee** — Enterprise, Google Cloud

### Backend for Frontend (BFF) Pattern

```
Web clients    ───► Web BFF    ───► Microservices
Mobile clients ───► Mobile BFF ───► Microservices
3rd Party      ───► Public API Gateway ──► Microservices

Each client type gets an optimized API
```

---

## 30. Idempotency

### What is it?
An operation is **idempotent** if executing it multiple times produces the same result as executing it once.

### Why It Matters

```
Scenario: Payment request
Client sends: POST /payments $100
Network timeout → Client doesn't know if it succeeded
Client retries: POST /payments $100

Without Idempotency: User charged $200 ❌
With Idempotency:    User charged $100 ✅
```

### HTTP Methods — Idempotency

```
✅ Idempotent:
   GET    /users/1        → Same response every time
   PUT    /users/1 {data} → Set to this state, repeat = same state
   DELETE /users/1        → Delete once; re-deleting = still deleted

❌ NOT Idempotent:
   POST   /orders         → Creates NEW order each time
   PATCH  /counter/inc    → Increments each time
```

### Implementing Idempotency Keys

```
Client generates unique key: UUID v4
Client sends:
  POST /payments
  Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000
  { "amount": 100 }

Server logic:
  1. Check if key exists in Redis/DB
  2. If YES → return cached response (don't process again)
  3. If NO  → process, store result with key → return response

Client retries with same key → same response ✅
```

### Idempotency Key Storage

```
┌────────┐   POST /pay            ┌────────────┐
│ Client │──── Idempotency-Key ──►│  Gateway   │
└────────┘    = "key-abc-123"     └─────┬──────┘
                                        │
                                  Check Redis
                                   ┌────┴─────┐
                                  HIT        MISS
                                   │           │
                               Return      Process
                               cached      payment,
                               result      cache result
```

### Design Patterns for Idempotency

```sql
-- 1. Database Unique Constraints:
INSERT INTO orders (idempotency_key, ...)
VALUES (...)
ON CONFLICT (idempotency_key) DO NOTHING;

-- 2. Conditional Updates (Optimistic Locking):
UPDATE accounts
SET balance = 100
WHERE version = 5;
```

```
3. Natural Idempotency:
   "Set user email to alice@example.com" is naturally idempotent
   Design operations as "set state" not "change state"
```

---

## 🗺️ Quick Reference — System Design Decision Guide

```
Scaling Problem?
├── Small scale → Vertical Scaling
├── More reads → Add Replicas + Caching
├── More writes → Horizontal Scaling + Sharding
└── Global users → CDN + Geographic Distribution

Database Choice?
├── Complex relations + ACID → SQL (PostgreSQL)
├── Flexible schema + high writes → MongoDB
├── Caching + sessions → Redis
├── Time-series / IoT → Cassandra
└── Social graph → Neo4j

Communication Pattern?
├── Request-Response → REST / GraphQL
├── Real-time bidirectional → WebSockets
├── Event notification → Webhooks
├── Async decoupling → Message Queues
└── High-performance internal → gRPC

Consistency vs Availability?
├── Banking / Inventory → CP (strong consistency)
└── Social feeds / DNS → AP (eventual consistency)

Performance Problem?
├── Slow DB reads → Indexing + Caching
├── Slow queries → Denormalization + Read replicas
├── High latency → CDN + Edge caching
└── DB overloaded → Sharding + Connection pooling
```

---

> 💡 **Interview Tip**: Always think in terms of **trade-offs**. Every system design decision involves balancing consistency vs availability, cost vs performance, simplicity vs scalability. Start simple, then scale — and always justify your choices.