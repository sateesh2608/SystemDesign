# Generic System Design Framework — Detailed Guide

This document provides a reusable framework for approaching almost any system design problem.

It is designed for:

- Senior Software Engineer interviews
- Backend and full-stack architecture discussions
- Technical lead and architect preparation
- Real-world solution design
- Structured system design documentation

The framework follows this progression:

```text
Requirements
    |
    v
Core Entities
    |
    v
API or Interface
    |
    v
Data Flow
    |
    v
High-Level Design
    |
    v
Deep Dives
```

The two primary goals are:

```text
1. Satisfy Functional Requirements
2. Satisfy Non-Functional Requirements
```

Functional requirements define **what the system must do**.

Non-functional requirements define **how well the system must do it**.

---

# Table of Contents

1. [System Design Overview](#1-system-design-overview)
2. [Step 1: Clarify Requirements](#2-step-1-clarify-requirements)
3. [Step 2: Identify Core Entities](#3-step-2-identify-core-entities)
4. [Step 3: Design APIs and Interfaces](#4-step-3-design-apis-and-interfaces)
5. [Step 4: Explain Data Flow](#5-step-4-explain-data-flow)
6. [Step 5: Build the High-Level Design](#6-step-5-build-the-high-level-design)
7. [Step 6: Deep-Dive into Critical Areas](#7-step-6-deep-dive-into-critical-areas)
8. [Capacity Estimation](#8-capacity-estimation)
9. [Database Design](#9-database-design)
10. [Caching Strategy](#10-caching-strategy)
11. [Asynchronous Processing](#11-asynchronous-processing)
12. [Scalability](#12-scalability)
13. [Reliability and Fault Tolerance](#13-reliability-and-fault-tolerance)
14. [Consistency and Transactions](#14-consistency-and-transactions)
15. [Security](#15-security)
16. [Observability](#16-observability)
17. [Performance Optimization](#17-performance-optimization)
18. [Common System Design Patterns](#18-common-system-design-patterns)
19. [Trade-Off Analysis](#19-trade-off-analysis)
20. [Common Interview Mistakes](#20-common-interview-mistakes)
21. [Generic Interview Walkthrough](#21-generic-interview-walkthrough)
22. [Reusable Answer Template](#22-reusable-answer-template)
23. [Interview Readiness Checklist](#23-interview-readiness-checklist)
24. [Senior Interview Summary](#24-senior-interview-summary)

---

# 1. System Design Overview

A system design discussion should begin with the problem, not with technologies.

A strong process is:

```text
Understand the problem
        |
        v
Define required behavior
        |
        v
Identify important data
        |
        v
Define interfaces
        |
        v
Explain critical flows
        |
        v
Build the architecture
        |
        v
Deep-dive into risk and scale
```

A weak answer starts like this:

```text
We will use microservices, Kafka, Redis, Kubernetes, and Cassandra.
```

That is technology-first thinking.

A stronger answer starts like this:

```text
First, I want to clarify user behavior, expected scale, latency,
availability, durability, consistency, and security. Based on those
requirements, I will select the architecture and technologies.
```

## 1.1 Functional Requirements

Functional requirements describe system behavior.

Examples:

- Users can create accounts
- Users can upload files
- Users can search for products
- Users can send messages
- Users can place orders
- Users can make payments
- Users can receive notifications

A functional requirement should answer:

```text
Who performs the action?
What action is performed?
What data is involved?
What result is expected?
```

Example:

```text
A customer can place an order containing one or more products.
```

## 1.2 Non-Functional Requirements

Non-functional requirements define quality attributes.

Common examples:

- Scalability
- Availability
- Reliability
- Durability
- Latency
- Throughput
- Consistency
- Security
- Maintainability
- Observability
- Cost efficiency
- Disaster recovery

Examples:

```text
The system should support 10 million daily active users.
```

```text
P95 search latency should remain below 200 milliseconds.
```

```text
Payment records must not be lost.
```

```text
The service should maintain 99.99% availability.
```

## 1.3 How the Two Goals Affect Design

Functional requirements mainly influence:

- Core entities
- APIs
- Business workflows
- Data relationships
- User interactions

Non-functional requirements mainly influence:

- Database selection
- Caching
- Replication
- Partitioning
- Load balancing
- Messaging
- Failover
- Multi-region design
- Security
- Monitoring

A complete design must satisfy both.

---

# 2. Step 1: Clarify Requirements

The first stage removes ambiguity.

Do not silently assume the scope.

Ask a few high-value questions before designing.

## 2.1 Clarify Scope

Determine what is included and excluded.

Example for a chat system:

### In Scope

- One-to-one messaging
- Group messaging
- Message delivery
- Online and offline handling

### Out of Scope Unless Requested

- Video calls
- End-to-end encryption
- Message search
- File attachments
- Message editing

Clear scope prevents unnecessary expansion.

## 2.2 Identify Actors

Actors are people, services, or external systems interacting with the design.

Examples:

- Customer
- Administrator
- Seller
- Delivery partner
- Payment provider
- Reporting system

```text
Customer
   |
   v
Order System
   |
   v
Payment Provider
```

Actors help identify:

- Interfaces
- permissions
- workflows
- external dependencies
- trust boundaries

## 2.3 Prioritize Features

Use:

```text
Must Have
Should Have
Nice to Have
Out of Scope
```

Example for a URL shortener:

### Must Have

- Create a short URL
- Redirect to the original URL
- Avoid short-code collisions

### Should Have

- Custom aliases
- Link expiration
- Basic analytics

### Nice to Have

- QR codes
- Geographic analytics

### Out of Scope

- Full marketing automation
- Advanced campaign management

## 2.4 Clarify Scale and Quality Targets

Ask:

- How many users?
- What is peak requests per second?
- Is the workload read-heavy or write-heavy?
- What is the average object size?
- What latency is acceptable?
- What availability is required?
- Can users tolerate stale data?
- What data must never be lost?
- Is the system regional or global?
- Are there compliance requirements?

## 2.5 Define Measurable Success

Examples:

- P95 latency below 200 ms
- 99.99% monthly availability
- Zero duplicate payment execution
- Search index freshness below 5 seconds
- Notifications delivered within 1 minute
- Recovery point objective below 5 minutes

## 2.6 Identify Constraints

Constraints may include:

- Existing cloud provider
- Existing databases
- Team expertise
- Budget
- Delivery timeline
- Compliance
- Data residency
- Legacy integration
- Vendor requirements

A practical design must respect organizational reality.

## 2.7 Requirement Template

```markdown
## Functional Requirements

1. Users can ...
2. Administrators can ...
3. External systems can ...

## Non-Functional Requirements

- Scale:
- Peak traffic:
- Latency:
- Availability:
- Durability:
- Consistency:
- Security:
- Geographic scope:

## Out of Scope

- ...
- ...
```

---

# 3. Step 2: Identify Core Entities

Core entities represent the main business objects.

They often become:

- Database tables
- Documents
- API resources
- Events
- Cache entries
- Aggregate roots

Examples:

| System | Core Entities |
|---|---|
| E-commerce | User, Product, Cart, Order, Payment |
| Chat | User, Conversation, Message, Participant |
| Ride Sharing | Rider, Driver, Ride, Location, Payment |
| Social Network | User, Post, Comment, Follow, Like |
| File Storage | User, File, Folder, Permission, Version |
| Notification System | User, Notification, Template, Preference |

## 3.1 Find Entities from Requirements

Look for nouns.

Requirement:

```text
A customer places an order containing products and pays using a payment method.
```

Likely entities:

- Customer
- Order
- Order Item
- Product
- Payment
- Payment Method

## 3.2 Define Relationships

Common relationship types:

- One-to-one
- One-to-many
- Many-to-many

Example:

```text
Customer 1 ------ N Order
Order    1 ------ N OrderItem
Product  1 ------ N OrderItem
Order    1 ------ 1 Payment
```

## 3.3 Example Entity Model

```text
User
- userId
- name
- email
- status
- createdAt

Order
- orderId
- userId
- status
- totalAmount
- createdAt

OrderItem
- orderItemId
- orderId
- productId
- quantity
- unitPrice

Payment
- paymentId
- orderId
- amount
- status
- providerReference
```

## 3.4 Questions for Each Entity

- What uniquely identifies it?
- Who owns it?
- How often is it created?
- How often is it updated?
- Is it mutable or immutable?
- How long is it retained?
- Is versioning required?
- Does it contain sensitive data?
- What relationships exist?
- Which queries access it most frequently?

Do not define every column and index yet. That belongs in a database deep dive.

---

# 4. Step 3: Design APIs and Interfaces

Interfaces define how clients and systems interact.

Possible interface types:

- REST
- GraphQL
- gRPC
- WebSockets
- Server-Sent Events
- Events
- Webhooks
- Batch files

Choose based on the communication requirement.

## 4.1 REST Example

```http
POST /api/v1/orders
```

Request:

```json
{
  "customerId": "cust-101",
  "items": [
    {
      "productId": "prod-501",
      "quantity": 2
    }
  ]
}
```

Response:

```http
201 Created
```

```json
{
  "orderId": "order-9001",
  "status": "CREATED",
  "totalAmount": 120.00
}
```

## 4.2 Main API Set

```http
POST   /api/v1/orders
GET    /api/v1/orders/{orderId}
GET    /api/v1/customers/{customerId}/orders
POST   /api/v1/orders/{orderId}/cancel
```

Domain commands such as cancellation may justify action-oriented endpoints when they involve business rules.

## 4.3 API Design Concerns

Include:

- Authentication
- Authorization
- Validation
- Idempotency
- Pagination
- Filtering
- Versioning
- Consistent errors
- Rate limiting
- Correlation IDs

## 4.4 Idempotency

Critical write operations should support idempotency.

```http
POST /payments
Idempotency-Key: payment-request-123
```

The server may store:

```text
Idempotency Key
Request Hash
Processing State
Stored Response
Expiration
```

A repeated request returns the original result rather than processing again.

## 4.5 Pagination

```http
GET /orders?limit=50&cursor=abc123
```

Cursor-based pagination is often preferred for large changing datasets.

## 4.6 Error Response

```json
{
  "timestamp": "2026-07-25T15:30:00Z",
  "status": 400,
  "code": "INVALID_ORDER",
  "message": "At least one order item is required",
  "correlationId": "corr-123"
}
```

## 4.7 Internal Interfaces

For internal high-throughput service communication:

```proto
service InventoryService {
  rpc ReserveInventory(ReserveInventoryRequest)
      returns (ReserveInventoryResponse);
}
```

Use gRPC when:

- strong contracts matter
- latency matters
- services call each other frequently
- streaming is required

Use REST when:

- simplicity matters
- APIs are external
- browser compatibility matters
- human readability is valuable

## 4.8 Event Contract

```json
{
  "eventId": "evt-1001",
  "eventType": "OrderCreated",
  "eventVersion": 1,
  "timestamp": "2026-07-25T15:30:00Z",
  "correlationId": "corr-123",
  "payload": {
    "orderId": "order-9001",
    "customerId": "cust-101"
  }
}
```

A production event should have:

- Event ID
- Event type
- Version
- Timestamp
- Correlation ID
- Payload

---

# 5. Step 4: Explain Data Flow

Explain one or two critical flows.

Examples:

- Create order
- Send message
- Upload file
- Search product
- Process payment
- Deliver notification

## 5.1 Synchronous Read Flow

```text
Client
   |
   v
Load Balancer
   |
   v
Order Service
   |
   v
Cache
   |
   +------ Hit ------> Response
   |
   +------ Miss -----> Database
                         |
                         v
                       Response
```

Explain:

1. Client sends request.
2. Load balancer routes it.
3. Service authenticates and validates.
4. Cache is checked.
5. Database is queried on a miss.
6. Response is returned.
7. Result may be cached.

## 5.2 Asynchronous Write Flow

```text
Client
   |
   v
Order API
   |
   v
Order Database
   |
   v
Outbox Event
   |
   v
Message Broker
   |
   +--------> Inventory Service
   |
   +--------> Payment Service
   |
   +--------> Notification Service
```

Explain:

1. Request is validated.
2. Business data is stored.
3. An outbox record is stored in the same transaction.
4. A publisher sends the event.
5. Consumers process independently.
6. Transient failures are retried.
7. Repeated failures move to a DLQ.

## 5.3 Read Path vs Write Path

### Write Path

```text
Client
  |
  v
Write API
  |
  v
Validation
  |
  v
Primary Database
  |
  v
Event Publication
```

### Read Path

```text
Client
  |
  v
Read API
  |
  v
Cache
  |
  v
Read Replica or Search Index
```

## 5.4 Include Failure Flow

Discuss:

- database unavailable
- cache unavailable
- downstream timeout
- duplicate request
- message processing failure
- server crash after commit
- response lost after successful processing

A production design must explain both the happy path and failure behavior.

---

# 6. Step 5: Build the High-Level Design

A generic web-scale architecture:

```text
Users
  |
  v
DNS
  |
  v
CDN / Edge
  |
  v
Load Balancer / API Gateway
  |
  v
Application Services
  |
  +----------> Cache
  |
  +----------> Database
  |
  +----------> Search Engine
  |
  +----------> Object Storage
  |
  +----------> Message Broker
                    |
                    v
              Background Workers
```

## 6.1 Client Layer

Clients may include:

- Web
- Mobile
- Partners
- Internal services
- IoT devices

Responsibilities:

- User interaction
- Authentication initiation
- Local validation
- Retry behavior
- User-visible error handling

## 6.2 DNS

DNS maps a domain name to a network endpoint.

Advanced uses:

- geographic routing
- failover
- weighted traffic
- multi-region routing

## 6.3 CDN

A CDN caches content close to users.

Use it for:

- images
- video
- JavaScript
- CSS
- static downloads
- public cacheable responses

Benefits:

- lower latency
- lower origin traffic
- better global performance
- DDoS absorption

## 6.4 Load Balancer

Responsibilities:

- TLS termination
- health checks
- traffic distribution
- failover
- path or host routing

Strategies:

- Round robin
- Least connections
- Weighted routing
- Path-based routing
- Host-based routing

## 6.5 API Gateway

An API gateway may provide:

- authentication
- rate limiting
- API keys
- quotas
- validation
- request transformation
- analytics

Do not add one automatically. A load balancer may be sufficient.

## 6.6 Application Services

Possible styles:

- Monolith
- Modular monolith
- Microservices
- Serverless functions

Example domains:

```text
Order Service
Payment Service
Inventory Service
Notification Service
```

A modular monolith may be better when:

- the team is small
- the domain is changing
- scale is moderate
- operational simplicity matters

## 6.7 Cache

Possible cache layers:

- Browser
- CDN
- API gateway
- Local application cache
- Distributed cache
- Database cache

Common cached data:

- sessions
- frequently read entities
- rate limits
- idempotency results
- feature flags
- computed responses

## 6.8 Database

Select based on access patterns.

Relational databases are strong for:

- transactions
- relationships
- constraints
- structured data
- strong consistency

NoSQL may be strong for:

- key-value access
- flexible documents
- massive scale
- partition-key-based workloads

## 6.9 Search Engine

Use for:

- full-text search
- fuzzy matching
- ranking
- autocomplete
- faceted search

The database remains the source of truth. The search index is often eventually consistent.

## 6.10 Object Storage

Use for:

- images
- videos
- documents
- backups
- large files

Store metadata in a database and binary content in object storage.

## 6.11 Message Broker

Use for:

- decoupling
- buffering
- event fan-out
- retries
- background processing

Examples include SQS, Kafka, RabbitMQ, and cloud pub/sub systems.

## 6.12 Background Workers

Workers may:

- send email
- resize images
- generate reports
- update search indexes
- process analytics
- reconcile payments

Workers should be:

- idempotent
- retryable
- observable
- horizontally scalable


# 7. Step 6: Deep-Dive into Critical Areas

After presenting the high-level design, choose the areas that carry the most risk or complexity.

Deep dives should be driven by:

- Expected scale
- Business criticality
- Consistency requirements
- Reliability requirements
- Security exposure
- Interviewer interest
- Known bottlenecks

Possible deep-dive areas:

- Database schema
- Partitioning and sharding
- Caching
- Search
- Messaging
- Distributed transactions
- Security
- Multi-region deployment
- Rate limiting
- Hot-key handling
- File upload
- Notification delivery

## 7.1 Select Deep Dives by System Type

### Payment System

Focus on:

- Idempotency
- Transaction integrity
- Ledger design
- Reconciliation
- Fraud controls
- Audit trail

### Chat System

Focus on:

- WebSocket scaling
- Message ordering
- Delivery states
- Offline delivery
- Fan-out
- Presence

### File Storage System

Focus on:

- Multipart upload
- Object storage
- Metadata
- Versioning
- Access control
- CDN delivery

### Search System

Focus on:

- Indexing pipeline
- Ranking
- Query latency
- Freshness
- Sharding
- Reindexing

## 7.2 Deep-Dive Structure

Use:

```text
Problem
  |
  v
Constraints
  |
  v
Alternatives
  |
  v
Chosen Approach
  |
  v
Trade-Offs
  |
  v
Failure Handling
```

---

# 8. Capacity Estimation

Capacity estimation gives direction to architecture decisions.

The goal is not perfect arithmetic. The goal is reasonable sizing.

## 8.1 User and Traffic Estimate

Example assumptions:

```text
Daily Active Users = 10 million
Requests per user per day = 20
```

Daily requests:

```text
10,000,000 × 20 = 200,000,000 requests/day
```

Average requests per second:

```text
200,000,000 / 86,400 ≈ 2,315 requests/second
```

If peak traffic is 8 times average:

```text
Peak ≈ 18,500 requests/second
```

## 8.2 Read/Write Ratio

Example:

```text
Read : Write = 90 : 10
Peak traffic = 20,000 requests/second
```

Then:

```text
Reads  = 18,000 requests/second
Writes = 2,000 requests/second
```

This may justify:

- caching
- read replicas
- separate read models
- search indexes

## 8.3 Storage Estimate

Example:

```text
1 million records/day
Average record size = 2 KB
```

Daily raw storage:

```text
1,000,000 × 2 KB = 2 GB/day
```

Annual raw storage:

```text
2 GB × 365 = 730 GB/year
```

Then include overhead for:

- indexes
- replicas
- backups
- metadata
- growth buffer

## 8.4 Bandwidth Estimate

Example:

```text
Peak responses = 10,000/second
Average response = 20 KB
```

```text
10,000 × 20 KB = 200 MB/second
```

This may justify:

- compression
- CDN
- regional deployment
- payload reduction

## 8.5 Concurrent Connections

For WebSockets, SSE, or long polling, estimate:

- Active connections
- Memory per connection
- File descriptors
- Heartbeat traffic
- Reconnect storms
- Connection distribution

Example:

```text
1 million concurrent WebSocket connections
```

If one server supports 50,000 healthy connections:

```text
Minimum servers = 20
```

Additional capacity is required for failover and peak traffic.

---

# 9. Database Design

Database choice should follow access patterns.

Start with:

```text
What are the critical reads and writes?
```

Do not start with:

```text
Which database is popular?
```

## 9.1 Relational Database

Use when:

- Transactions matter
- Relationships matter
- Constraints matter
- Structured queries are required
- Strong consistency is important

Examples:

- PostgreSQL
- MySQL
- Oracle
- SQL Server

## 9.2 NoSQL Categories

### Key-Value

Best for:

- Sessions
- Counters
- Simple lookups
- Cache data

### Document

Best for:

- Flexible JSON structures
- Product catalogs
- User profiles
- Content metadata

### Wide Column

Best for:

- Large-scale partitioned data
- Time-series-like access
- High write volume
- Query-by-partition-key workloads

### Graph

Best for:

- Social relationships
- Recommendation paths
- Fraud networks
- Dependency graphs

## 9.3 Primary Key Selection

Options:

| ID Type | Advantage | Limitation |
|---|---|---|
| Auto increment | Small and ordered | Harder across distributed writers |
| UUID | Globally unique | Larger indexes and random insertion |
| Time-sortable distributed ID | Unique and ordered | More implementation complexity |
| Composite key | Matches access pattern | Harder references and migration |

Choose a key that is:

- Unique
- Stable
- Efficient to index
- Suitable for distribution

## 9.4 Indexing

Indexes improve reads but increase:

- Storage
- Write cost
- Maintenance
- Operational complexity

Example query:

```sql
SELECT *
FROM orders
WHERE customer_id = ?
ORDER BY created_at DESC;
```

Possible index:

```text
(customer_id, created_at DESC)
```

Design indexes around actual query patterns.

## 9.5 Normalization vs Denormalization

Normalization:

- Reduces duplication
- Improves consistency
- Works well for transactions

Denormalization:

- Reduces joins
- Speeds reads
- Duplicates data
- Complicates updates

Use denormalization intentionally for high-value read paths.

## 9.6 Replication

```text
Primary
   |
   +----> Read Replica 1
   |
   +----> Read Replica 2
```

Benefits:

- Read scaling
- High availability
- Disaster recovery

Challenges:

- Replication lag
- Stale reads
- Failover complexity
- Write conflicts in multi-primary designs

## 9.7 Partitioning and Sharding

Possible partition keys:

- User ID
- Tenant ID
- Region
- Time
- Hash of entity ID

A good key has:

- High cardinality
- Even distribution
- Alignment with access patterns
- Minimal cross-partition queries

Poor keys create hot partitions.

## 9.8 Transactions

Use transactions when multiple changes must succeed or fail together.

Within one database, use ACID transactions.

Across services, consider:

- Saga
- Transactional outbox
- Compensating actions
- Idempotency
- Reconciliation

---

# 10. Caching Strategy

Caching improves latency and reduces backend load, but it introduces consistency complexity.

## 10.1 Cache-Aside

```text
Application
   |
   | Check
   v
Cache
   |
   +-- Hit --> Return
   |
   +-- Miss --> Database
                  |
                  v
               Fill Cache
```

Advantages:

- Simple
- Application controlled
- Common

Risks:

- Stale values
- Cache stampede
- Cold start

## 10.2 Write-Through

```text
Application
   |
   v
Cache
   |
   v
Database
```

Advantage:

- Cache remains warm

Limitation:

- Higher write latency

## 10.3 Write-Behind

Writes are accepted in cache and persisted later.

Advantages:

- Low write latency
- Batching

Risks:

- Data loss if cache fails
- More difficult durability
- Complex recovery

## 10.4 Invalidation

Methods:

- TTL
- Explicit deletion
- Event-driven invalidation
- Versioned keys

Example:

```text
customer:101:v5
```

## 10.5 Cache Stampede

Many requests miss the same key and hit the database simultaneously.

Mitigations:

- Request coalescing
- Distributed locks
- Early refresh
- Randomized TTL
- Stale-while-revalidate

## 10.6 Hot Keys

One very popular key can overload a cache node.

Mitigations:

- Local caching
- Key replication
- CDN
- Request aggregation
- Precomputation
- Partition-aware duplication

## 10.7 Cache Failure

Decide whether the application should:

- Fall back to database
- Return degraded data
- Reject traffic
- Use local cache
- Apply rate limits

The database must be protected from a sudden full cache miss.

---

# 11. Asynchronous Processing

Use asynchronous processing when the client does not need immediate completion.

Examples:

- Email delivery
- Report generation
- Image resizing
- Search indexing
- Analytics
- Payment reconciliation

## 11.1 Queue Architecture

```text
Producer
   |
   v
Queue or Topic
   |
   v
Consumer
```

Benefits:

- Decoupling
- Buffering
- Independent scaling
- Failure isolation
- Retry support

## 11.2 Delivery Semantics

### At-Most-Once

May lose messages but avoids duplicate delivery.

### At-Least-Once

Avoids message loss but may deliver duplicates.

### Exactly-Once

Difficult to guarantee end to end.

A practical pattern is:

```text
At-least-once delivery
+
Idempotent consumer
=
Effectively-once business result
```

## 11.3 Idempotent Consumer

Techniques:

- Processed-event table
- Unique constraint
- Idempotency key
- State-transition validation
- Compare-and-set
- Deduplication cache

## 11.4 Retry

Use:

- Limited attempts
- Exponential backoff
- Jitter
- Retry only transient failures

Do not retry:

- Invalid input
- Authorization failure
- Permanent business rejection

## 11.5 Dead-Letter Queue

A DLQ stores repeatedly failing messages.

A complete DLQ process includes:

- Alerting
- Inspection
- Correction
- Replay
- Audit

## 11.6 Transactional Outbox

Problem:

```text
Database commit succeeds
Event publication fails
```

Solution:

1. Store business data.
2. Store outbox event in the same database transaction.
3. Publisher reads the outbox.
4. Publisher sends to broker.
5. Outbox record is marked as published.

This avoids unsafe dual writes.

## 11.7 Backpressure

If consumers are slower than producers:

- Queue depth grows
- Message age increases
- Latency increases

Possible responses:

- Scale consumers
- Slow producers
- Batch work
- Prioritize messages
- Shed noncritical work
- Increase partition count where supported

---

# 12. Scalability

Scalability is the ability to handle growth without unacceptable degradation.

## 12.1 Vertical Scaling

Increase resources on one machine:

- CPU
- Memory
- Storage
- Network capacity

Advantages:

- Simple

Limitations:

- Hardware ceiling
- Higher cost
- Single-machine risk

## 12.2 Horizontal Scaling

Add more instances.

```text
Load Balancer
   |
   +----> Instance 1
   +----> Instance 2
   +----> Instance 3
```

Requirements:

- Stateless application nodes
- Shared state outside process
- Distributed coordination only where necessary

## 12.3 Stateless Services

Store shared state in:

- Database
- Distributed cache
- Object storage
- Signed token

Benefits:

- Easy load balancing
- Easy failover
- Flexible scaling

## 12.4 Database Scaling Order

A practical progression:

```text
Optimize queries
      |
      v
Add indexes
      |
      v
Add caching
      |
      v
Add read replicas
      |
      v
Partition
      |
      v
Shard
```

Sharding should not be the first response to a slow query.

## 12.5 Autoscaling Signals

Use metrics such as:

- CPU
- Memory
- Requests per second
- P95 latency
- Queue depth
- Oldest message age
- Active connections

For workers, queue depth and message age are often more useful than CPU alone.

## 12.6 Hot Partition Handling

Options:

- Better partition key
- Salting
- Key splitting
- Dedicated partitions
- Replication
- Local cache
- Load-aware routing

---

# 13. Reliability and Fault Tolerance

Reliability means the system behaves correctly over time.

Fault tolerance means the system continues operating when components fail.

## 13.1 Redundancy

Avoid single points of failure.

Use:

- Multiple application instances
- Multi-zone deployment
- Replicated databases
- Redundant message brokers
- Redundant load balancers

## 13.2 Health Checks

### Liveness

Is the process alive?

### Readiness

Can the instance safely serve traffic?

A service may be alive but not ready if:

- Startup is incomplete
- Required configuration is missing
- A critical dependency is unavailable

## 13.3 Timeouts

Every network call should have a timeout.

Without timeouts:

- Threads block
- Connections accumulate
- Resources exhaust
- Failures cascade

## 13.4 Retries

Retries should be:

- Bounded
- Delayed
- Randomized
- Used only for safe or idempotent operations

Retry storms can worsen outages.

## 13.5 Circuit Breaker

```text
CLOSED
   |
   | failure threshold reached
   v
OPEN
   |
   | wait period
   v
HALF_OPEN
   |
   +-- success --> CLOSED
   |
   +-- failure --> OPEN
```

Benefits:

- Fail fast
- Protect dependencies
- Reduce blocked resources
- Allow recovery

## 13.6 Bulkhead

Separate resources so one failure cannot consume everything.

Examples:

- Separate thread pools
- Separate connection pools
- Separate queues
- Per-tenant limits

## 13.7 Graceful Degradation

Examples:

- Return cached recommendations
- Disable analytics
- Queue notifications
- Hide noncritical widgets
- Return basic search instead of personalized search

## 13.8 Disaster Recovery

### RPO

Recovery Point Objective: acceptable data loss.

### RTO

Recovery Time Objective: acceptable recovery time.

Example:

```text
RPO = 5 minutes
RTO = 30 minutes
```

---

# 14. Consistency and Transactions

Distributed systems require consistency trade-offs.

## 14.1 Strong Consistency

A successful write is immediately visible to subsequent reads.

Useful for:

- Payments
- Account balances
- Inventory reservation
- Authorization

## 14.2 Eventual Consistency

Replicas or derived systems converge over time.

Useful for:

- Feeds
- Search indexes
- Analytics
- Recommendations
- Counters

## 14.3 Read-Your-Writes

A user should see their recent update.

Possible techniques:

- Temporarily read from primary
- Update cache immediately
- Session-based routing
- Version token
- Monotonic read policy

## 14.4 Saga Pattern

```text
Create Order
   |
   v
Reserve Inventory
   |
   v
Charge Payment
   |
   v
Confirm Order
```

If payment fails:

```text
Release Inventory
Cancel Order
```

Saga styles:

- Choreography
- Orchestration

## 14.5 Optimistic Locking

Use a version field.

```text
Order
- orderId
- status
- version
```

Update succeeds only when the expected version matches.

Best when conflicts are uncommon.

## 14.6 Pessimistic Locking

Lock data before modification.

Useful when conflicts are frequent.

Trade-offs:

- Lower concurrency
- Lock waits
- Deadlock risk

## 14.7 Reconciliation

Distributed operations may require scheduled reconciliation.

Examples:

- Compare internal payment state with provider state
- Repair missing events
- Recalculate balances
- Detect orphaned records

Reconciliation is a critical safety mechanism for financial and distributed workflows.


# 15. Security

Security should be part of the architecture from the beginning.

## 15.1 Authentication

Authentication verifies identity.

Common methods:

- Username and password
- OAuth 2.0
- OpenID Connect
- JWT access token
- API key
- Mutual TLS
- Cloud workload identity

## 15.2 Authorization

Authorization determines what an authenticated identity may do.

Common models:

- Role-Based Access Control
- Attribute-Based Access Control
- Resource ownership
- Policy-based access
- Tenant-based isolation

Always authorize the specific resource.

A valid token alone is not enough.

Example:

```http
GET /customers/102/orders
```

The service must verify that the caller is allowed to access customer 102.

## 15.3 Encryption

### In Transit

Use TLS for network communication.

### At Rest

Encrypt:

- Databases
- Object storage
- Backups
- Sensitive logs
- Message payloads where required

## 15.4 Secrets Management

Do not store secrets in:

- Source code
- Public configuration
- Container images
- Logs
- Client-side applications

Use:

- Cloud secret managers
- Vault
- Workload identity
- Short-lived credentials
- Automatic rotation

## 15.5 Rate Limiting

Rate limiting protects against:

- Abuse
- Brute-force attempts
- Expensive requests
- Accidental client loops
- Resource exhaustion

Common algorithms:

- Fixed window
- Sliding window
- Token bucket
- Leaky bucket

## 15.6 Input Validation

Validate:

- Type
- Size
- Format
- Range
- Ownership
- Business rules

Protect against:

- SQL injection
- Command injection
- Cross-site scripting
- Path traversal
- Malicious file uploads
- Oversized payloads

## 15.7 Audit Logging

Record sensitive actions:

- Login
- Permission changes
- Payment actions
- Account deletion
- Data export
- Administrative operations

Audit logs should be:

- Tamper-resistant
- Searchable
- Access controlled
- Retained according to policy

## 15.8 Multi-Tenant Security

For multi-tenant systems:

- Include tenant identity in every request context
- Apply tenant-aware authorization
- Partition or filter data by tenant
- Avoid cross-tenant cache keys
- Include tenant context in logs and metrics
- Test for tenant data leakage

---

# 16. Observability

Observability helps operators understand internal behavior from external signals.

The three primary pillars are:

- Logs
- Metrics
- Traces

## 16.1 Structured Logs

Logs should include:

- Timestamp
- Severity
- Service
- Environment
- Correlation ID
- Request ID
- Error code
- Safe user or tenant identifier
- Relevant business identifier

Avoid logging:

- Passwords
- Access tokens
- Full card numbers
- Secrets
- Sensitive personal data

## 16.2 Metrics

Important service metrics:

- Request rate
- Error rate
- P50, P95, and P99 latency
- CPU
- Memory
- Thread pool usage
- Connection pool usage
- Active request count

Dependency metrics:

- Database latency
- Cache hit ratio
- Queue depth
- Oldest message age
- Downstream error rate
- Replica lag

## 16.3 Distributed Tracing

Tracing follows one request across services.

```text
Client
  |
  v
Order Service
  |
  v
Inventory Service
  |
  v
Payment Service
```

A trace helps identify:

- Slow service
- Slow database call
- Failed dependency
- Excessive retries
- Unexpected fan-out

## 16.4 Correlation IDs

A correlation ID connects:

- API logs
- Service logs
- Events
- Traces
- Error responses

Pass it across synchronous calls and asynchronous messages.

## 16.5 SLI, SLO, and SLA

### SLI

A measured indicator.

Example:

```text
Percentage of successful requests
```

### SLO

An internal reliability target.

Example:

```text
99.95% successful requests per month
```

### SLA

An external contractual commitment.

## 16.6 Alerting

Good alerts are actionable.

Examples:

- Error rate exceeds threshold for 10 minutes
- P95 latency violates SLO
- Queue backlog continues to grow
- Oldest message age is too high
- Database storage is near capacity
- Replica lag is above safe limit
- Regional health check fails

Avoid creating alerts for every temporary fluctuation.

---

# 17. Performance Optimization

Performance work should be measurement-driven.

## 17.1 Common Bottlenecks

- Slow database queries
- Missing indexes
- N+1 database queries
- Excessive service calls
- Network latency
- Large payloads
- Serialization overhead
- Lock contention
- Thread exhaustion
- Connection pool exhaustion
- Cache stampede
- Hot partitions

## 17.2 Optimization Process

```text
Measure
  |
  v
Identify Bottleneck
  |
  v
Apply Targeted Fix
  |
  v
Measure Again
```

Do not optimize only from assumptions.

## 17.3 Common Improvements

- Database indexes
- Query optimization
- Caching
- Pagination
- Batching
- Compression
- Asynchronous processing
- Connection pooling
- Precomputation
- CDN
- Payload reduction
- Read replicas
- Avoiding N+1 calls

## 17.4 Percentile Latency

Average latency can hide slow requests.

Example:

```text
P50 = 80 ms
P95 = 200 ms
P99 = 900 ms
```

The P99 shows tail-latency problems that the average may hide.

## 17.5 Latency Budget

Break the total target into components.

Example:

```text
Total P95 target = 300 ms

Load balancer        10 ms
Authentication       20 ms
Application logic    50 ms
Database            120 ms
Downstream service   70 ms
Network and buffer   30 ms
```

A latency budget makes optimization decisions more concrete.

---

# 18. Common System Design Patterns

## 18.1 Load Balancer

Distributes traffic across healthy instances.

## 18.2 Cache-Aside

Application reads from cache first and loads missing values from the database.

## 18.3 Message Queue

Decouples producers and consumers.

## 18.4 Publish-Subscribe

One event is delivered to multiple interested consumers.

## 18.5 Event-Driven Architecture

Services react to domain events.

## 18.6 Circuit Breaker

Stops repeated calls to a failing dependency.

## 18.7 Retry with Backoff

Retries temporary failures while reducing overload risk.

## 18.8 Bulkhead

Separates resources so one failure does not consume all capacity.

## 18.9 Saga

Coordinates distributed business transactions through local transactions and compensation.

## 18.10 Transactional Outbox

Publishes events reliably after database changes.

## 18.11 CQRS

Separates command and query models when reads and writes have very different requirements.

## 18.12 Sharding

Distributes data across partitions.

## 18.13 Leader-Follower Replication

One node accepts writes while followers replicate and may serve reads.

## 18.14 Consistent Hashing

Distributes keys while minimizing movement when nodes join or leave.

## 18.15 Idempotency

Ensures retries do not produce duplicate business outcomes.

## 18.16 Materialized View

Precomputes a read-optimized representation.

Useful for:

- Dashboards
- Aggregations
- Search
- Reporting

## 18.17 Rate Limiter

Controls how much traffic a caller may generate.

## 18.18 Strangler Pattern

Gradually replaces a legacy system by routing selected capabilities to new services.

---

# 19. Trade-Off Analysis

Every major design choice has trade-offs.

A strong answer states them explicitly.

## 19.1 SQL vs NoSQL

### SQL

Advantages:

- Transactions
- Relationships
- Constraints
- Rich query support

Limitations:

- Horizontal write scaling can be complex
- Schema evolution may require coordination

### NoSQL

Advantages:

- Flexible models
- High horizontal scale
- Access-pattern optimization

Limitations:

- Fewer relational guarantees
- Denormalization
- Application-managed consistency in some systems

## 19.2 Synchronous vs Asynchronous

### Synchronous

Advantages:

- Immediate response
- Simpler request flow
- Easy client understanding

Limitations:

- Tight runtime coupling
- Cascading-failure risk
- Higher latency across multiple services

### Asynchronous

Advantages:

- Decoupling
- Buffering
- Independent scaling
- Failure isolation

Limitations:

- Eventual consistency
- Harder debugging
- Duplicate processing
- More operational components

## 19.3 Strong vs Eventual Consistency

### Strong

Advantages:

- Simple user expectations
- Immediate correctness

Limitations:

- More coordination
- Higher latency
- Reduced availability during partitions in some designs

### Eventual

Advantages:

- Better availability
- Better scale
- Lower coordination cost

Limitations:

- Stale reads
- Reconciliation
- More complex user experience

## 19.4 Monolith vs Microservices

### Monolith or Modular Monolith

Advantages:

- Simpler deployment
- Easier transactions
- Easier local development
- Lower operational cost

Limitations:

- Larger deployment unit
- Coupling can grow
- Independent scaling is harder

### Microservices

Advantages:

- Independent deployment
- Domain ownership
- Independent scaling
- Technology flexibility

Limitations:

- Distributed transactions
- Network failures
- Observability complexity
- Higher operational cost

## 19.5 Build vs Buy

### Build

Advantages:

- Full control
- Custom behavior
- No platform dependency

Limitations:

- Engineering time
- Maintenance burden
- Operational ownership

### Buy

Advantages:

- Faster delivery
- Mature features
- Vendor support

Limitations:

- Recurring cost
- Vendor dependency
- Integration constraints
- Migration difficulty

## 19.6 Availability vs Consistency

During a network partition, some distributed systems must choose between:

- Rejecting requests to preserve consistency
- Serving requests with potentially stale or divergent data

The correct choice depends on the business domain.

---

# 20. Common Interview Mistakes

## 20.1 Starting with Technology

Weak:

```text
I will use Kafka, Cassandra, Redis, and Kubernetes.
```

Strong:

```text
Because the system has high write volume, asynchronous fan-out, and
eventual-consistency tolerance, I would introduce a durable message broker.
```

## 20.2 Ignoring Requirements

Do not design before clarifying:

- Scope
- Scale
- Latency
- Availability
- Durability
- Consistency
- Security

## 20.3 Overengineering

Do not add:

- Microservices
- Kafka
- Sharding
- Multi-region active-active
- Service mesh
- CQRS

unless the requirements justify them.

## 20.4 Ignoring Failure Cases

Discuss:

- Downstream timeout
- Database outage
- Cache outage
- Duplicate request
- Duplicate event
- Partial transaction
- Regional outage

## 20.5 Missing Trade-Offs

For every important decision, answer:

```text
Why this choice?
What alternative exists?
What is the downside?
```

## 20.6 Going Too Deep Too Early

Start with the complete high-level design.

Deep-dive only after the major flow is understandable.

## 20.7 Designing Every Feature

Prioritize the most important use cases.

Do not spend the interview designing optional features before core requirements.

## 20.8 Ignoring Operational Concerns

A production-ready design should cover:

- Deployment
- Monitoring
- Alerting
- Failure recovery
- Backups
- Security
- Cost

## 20.9 Not Managing Time

Example allocation for a 45-minute interview:

```text
Requirements         5 minutes
Entities and APIs    5 minutes
Data Flow            5 minutes
High-Level Design   10 minutes
Deep Dives          15 minutes
Trade-Offs           5 minutes
```

---

# 21. Generic Interview Walkthrough

Suppose the question is:

```text
Design an online order management system.
```

## 21.1 Requirements

### Functional

- Customer creates an order
- Customer views order
- Customer cancels an eligible order
- System processes payment
- System reserves inventory
- System sends notifications

### Non-Functional

- 5 million daily users
- 99.99% availability
- P95 latency below 300 ms
- No duplicate payment execution
- Order data must be durable
- Search and reporting may be eventually consistent

## 21.2 Core Entities

```text
Customer
Product
Order
OrderItem
Payment
InventoryReservation
Notification
```

## 21.3 APIs

```http
POST /orders
GET /orders/{orderId}
POST /orders/{orderId}/cancel
GET /customers/{customerId}/orders
```

Payment creation uses an idempotency key.

## 21.4 Data Flow

```text
Client
  |
  v
Order Service
  |
  v
Order Database + Outbox
  |
  v
Message Broker
  |
  +----> Inventory Service
  |
  +----> Payment Service
  |
  +----> Notification Service
```

## 21.5 High-Level Design

```text
Web / Mobile
    |
    v
Load Balancer
    |
    v
Order Service
    |
    +----> Redis
    |
    +----> PostgreSQL
    |
    +----> Message Broker
               |
               +----> Inventory Worker
               +----> Payment Worker
               +----> Notification Worker
```

## 21.6 Important Deep Dives

- Payment idempotency
- Inventory reservation
- Saga and compensation
- Transactional outbox
- Retry and DLQ
- Order-status consistency
- Database indexes and partitioning
- Audit logs
- Observability

## 21.7 Trade-Off Statement

```text
I would keep order creation strongly consistent within the Order Service,
but allow notification delivery and search indexing to be eventually
consistent. I would use asynchronous events to decouple downstream work,
while using idempotency and reconciliation to handle duplicate or missed
processing safely.
```

---

# 22. Reusable Answer Template

```markdown
# System Name

## 1. Scope

### In Scope

- ...
- ...

### Out of Scope

- ...
- ...

## 2. Functional Requirements

1. ...
2. ...
3. ...

## 3. Non-Functional Requirements

- Scale:
- Peak traffic:
- Latency:
- Availability:
- Durability:
- Consistency:
- Security:
- Geographic scope:

## 4. Capacity Estimates

- Daily active users:
- Average RPS:
- Peak RPS:
- Read/write ratio:
- Storage growth:
- Bandwidth:
- Concurrent connections:

## 5. Core Entities

- Entity 1
- Entity 2
- Entity 3

## 6. APIs or Interfaces

```http
POST /...
GET /...
```

## 7. Main Data Flow

```text
Client
  |
  v
...
```

## 8. High-Level Architecture

```text
...
```

## 9. Database Design

- Database choice:
- Main tables:
- Primary keys:
- Indexes:
- Replication:
- Partitioning:

## 10. Caching

- Cached data:
- Cache pattern:
- TTL:
- Invalidation:
- Failure behavior:

## 11. Asynchronous Processing

- Broker:
- Events:
- Consumers:
- Retry:
- DLQ:
- Idempotency:

## 12. Scalability

- Stateless services:
- Autoscaling:
- Read scaling:
- Write scaling:
- Hot-partition handling:

## 13. Reliability

- Redundancy:
- Timeouts:
- Retries:
- Circuit breakers:
- Failover:
- Disaster recovery:

## 14. Consistency

- Strongly consistent data:
- Eventually consistent data:
- Saga:
- Reconciliation:

## 15. Security

- Authentication:
- Authorization:
- Encryption:
- Secrets:
- Rate limiting:
- Audit:

## 16. Observability

- Logs:
- Metrics:
- Traces:
- Alerts:
- SLOs:

## 17. Trade-Offs

- Decision:
- Alternative:
- Reason:
- Limitation:

## 18. Future Improvements

- ...
- ...
```

---

# 23. Interview Readiness Checklist

## Requirements

- [ ] I clarify functional requirements.
- [ ] I clarify non-functional requirements.
- [ ] I identify actors.
- [ ] I define scope.
- [ ] I define out-of-scope features.
- [ ] I identify scale and latency expectations.
- [ ] I clarify consistency and availability requirements.

## Core Entities

- [ ] I identify major business entities.
- [ ] I explain relationships.
- [ ] I identify ownership.
- [ ] I identify sensitive data.
- [ ] I avoid premature low-level schema design.

## APIs

- [ ] I define the main APIs.
- [ ] I use appropriate methods and status codes.
- [ ] I explain authentication and authorization.
- [ ] I include idempotency for critical writes.
- [ ] I include pagination where needed.
- [ ] I design consistent error responses.
- [ ] I define event contracts when needed.

## Data Flow

- [ ] I explain one critical write flow.
- [ ] I explain one critical read flow.
- [ ] I distinguish synchronous and asynchronous steps.
- [ ] I identify the source of truth.
- [ ] I explain failure behavior.

## High-Level Design

- [ ] I include clients and entry points.
- [ ] I include load balancing.
- [ ] I define service boundaries.
- [ ] I include the database and cache.
- [ ] I include messaging only when justified.
- [ ] I include search or object storage only when justified.
- [ ] I avoid unnecessary components.

## Capacity

- [ ] I estimate users.
- [ ] I estimate average and peak RPS.
- [ ] I estimate read/write ratio.
- [ ] I estimate storage.
- [ ] I estimate bandwidth.
- [ ] I estimate concurrent connections where relevant.

## Database

- [ ] I choose the database based on access patterns.
- [ ] I explain keys and indexes.
- [ ] I discuss replication.
- [ ] I discuss partitioning only when needed.
- [ ] I discuss consistency and transactions.

## Cache

- [ ] I explain what is cached.
- [ ] I explain invalidation.
- [ ] I define TTL.
- [ ] I discuss cache stampede.
- [ ] I discuss hot keys.
- [ ] I explain cache failure behavior.

## Messaging

- [ ] I explain why messaging is needed.
- [ ] I define events.
- [ ] I explain delivery semantics.
- [ ] I explain retries and DLQ.
- [ ] I explain idempotent consumers.
- [ ] I explain transactional outbox when applicable.
- [ ] I discuss backpressure.

## Scalability

- [ ] I design stateless services.
- [ ] I explain horizontal scaling.
- [ ] I explain database read scaling.
- [ ] I explain database write scaling.
- [ ] I discuss autoscaling signals.
- [ ] I handle hot partitions.

## Reliability

- [ ] I remove single points of failure.
- [ ] I use health checks.
- [ ] I define timeouts.
- [ ] I define retry policies.
- [ ] I discuss circuit breakers.
- [ ] I discuss graceful degradation.
- [ ] I discuss disaster recovery.

## Security

- [ ] I explain authentication.
- [ ] I explain resource-level authorization.
- [ ] I use encryption in transit and at rest.
- [ ] I use secrets management.
- [ ] I include rate limiting.
- [ ] I include audit logging.
- [ ] I consider tenant isolation.

## Observability

- [ ] I include structured logs.
- [ ] I include metrics.
- [ ] I include distributed tracing.
- [ ] I define alerts.
- [ ] I define SLOs.
- [ ] I use correlation IDs.

## Trade-Offs

- [ ] I explain why each major component exists.
- [ ] I mention alternatives.
- [ ] I discuss disadvantages.
- [ ] I avoid claiming one technology is always best.
- [ ] I keep the design aligned with requirements.

---

# 24. Senior Interview Summary

> I approach system design in a structured sequence. I first clarify functional and non-functional requirements, define scope, identify expected scale, and understand latency, availability, durability, consistency, and security expectations. I then identify the core business entities and relationships because they drive the data model and service boundaries. Next, I define the main APIs or interfaces and walk through the critical read and write data flows. After that, I build the high-level architecture using only the components justified by the requirements, such as load balancers, stateless services, caches, databases, search engines, object storage, and message brokers. I then deep-dive into the highest-risk areas, such as database scaling, idempotency, distributed transactions, caching, asynchronous processing, consistency, reliability, and security. Throughout the design, I explain alternatives and trade-offs rather than presenting one technology as universally correct. Finally, I cover observability, failure recovery, and future scaling so the design is not only functional but also production-ready.
