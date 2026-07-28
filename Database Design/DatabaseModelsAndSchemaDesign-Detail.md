# Database Models and Schema Design — Detailed Interview Guide

This guide provides a senior-level understanding of:

- Relational databases
- Document databases
- Key-value stores
- Wide-column databases
- Graph databases
- Schema design fundamentals
- Keys and relationships
- Access-pattern-driven indexing
- Normalization and denormalization
- Replication, partitioning, scaling, and sharding
- Database selection for system design interviews

It is written for experienced backend engineers who must explain not only definitions, but also trade-offs, operational implications, failure modes, and production design decisions.

For a concise overview, see [Database Models and Schema Design](DatabaseModelsAndSchemaDesign.md).

---

# Table of Contents

1. [How Senior Engineers Choose a Database](#1-how-senior-engineers-choose-a-database)
2. [Requirements and Workload Analysis](#2-requirements-and-workload-analysis)
3. [Schema Design Process](#3-schema-design-process)
4. [Entities, Aggregates, Keys, and Relationships](#4-entities-aggregates-keys-and-relationships)
5. [Relational Databases](#5-relational-databases)
6. [Document Databases](#6-document-databases)
7. [Key-Value Stores](#7-key-value-stores)
8. [Wide-Column Databases](#8-wide-column-databases)
9. [Graph Databases](#9-graph-databases)
10. [Access-Pattern-Driven Design](#10-access-pattern-driven-design)
11. [Indexing](#11-indexing)
12. [Normalization](#12-normalization)
13. [Denormalization](#13-denormalization)
14. [Transactions and Consistency](#14-transactions-and-consistency)
15. [Replication](#15-replication)
16. [Partitioning and Sharding](#16-partitioning-and-sharding)
17. [Hot Partitions and Skew](#17-hot-partitions-and-skew)
18. [Schema Evolution and Migration](#18-schema-evolution-and-migration)
19. [Polyglot Persistence](#19-polyglot-persistence)
20. [Security and Governance](#20-security-and-governance)
21. [Observability and Operations](#21-observability-and-operations)
22. [Scenario-Based Selection](#22-scenario-based-selection)
23. [Trade-Off Matrix](#23-trade-off-matrix)
24. [Common Interview Questions](#24-common-interview-questions)
25. [Interview Readiness Tracker](#25-interview-readiness-tracker)
26. [Senior Interview Answer](#26-senior-interview-answer)

---

# 1. How Senior Engineers Choose a Database

A senior engineer should not choose a database based on popularity or one attractive feature.

Use this sequence:

```text
Business Requirements
        |
        v
Data Shape and Relationships
        |
        v
Read and Write Access Patterns
        |
        v
Consistency and Transaction Needs
        |
        v
Scale and Availability
        |
        v
Operational Constraints
        |
        v
Database Model and Product
```

Weak answer:

```text
MongoDB is schema-less and scales, so I would use MongoDB.
```

Stronger answer:

```text
The product catalog contains highly variable attributes, is read far more
often than it is written, and is usually retrieved as a complete aggregate.
A document model fits because it avoids many joins and supports category-
specific attributes. I would still enforce validation, define document size
limits, and maintain a search index for full-text and faceted queries.
```

A complete database decision explains:

- Why the model fits
- Which queries it serves
- How data remains correct
- How it scales
- How it fails
- What operational burden it adds
- Which alternative was rejected

---

# 2. Requirements and Workload Analysis

Before schema design, clarify the workload.

## 2.1 Functional Questions

- What does the system store?
- Who creates the data?
- Who owns the data?
- Which workflows modify it?
- Which records are immutable?
- Which data is shared?
- Which entities require history?
- Which queries must be supported?
- Which reports or searches are required?

## 2.2 Non-Functional Questions

- Peak reads per second
- Peak writes per second
- Read/write ratio
- Data size and growth
- Retention period
- Latency targets
- Availability target
- Durability requirement
- Regional distribution
- Multi-tenancy
- Compliance
- Backup and recovery
- Cost constraints

## 2.3 Consistency Questions

- Must every read see the latest write?
- Can replicas return stale data?
- Must multiple records update atomically?
- Are cross-entity transactions required?
- Can the business use compensation?
- Is read-your-writes consistency enough?
- Can derived views be eventually consistent?
- Is conflict resolution required across regions?

## 2.4 Query Questions

List the top queries before designing tables.

Example:

```text
Get an order by order ID.

Get the latest 50 orders for a customer.

Find all orders in PROCESSING state older than 30 minutes.

Find payments by provider reference.

Aggregate daily sales by region.

Search products by text, category, and price.
```

Each access pattern may affect:

- Primary key
- Secondary index
- Table design
- Document boundary
- Partition key
- Sort key
- Materialized view
- Search index

## 2.5 Read and Write Characteristics

Determine whether writes are:

- Insert-only
- Update-heavy
- Delete-heavy
- Batch-oriented
- Counter-based
- Conditional
- Transactional
- Append-only

Determine whether reads are:

- Exact-key lookup
- Range scan
- Full-text search
- Aggregation
- Multi-hop traversal
- Multi-entity join
- Large sequential scan

---

# 3. Schema Design Process

Use this sequence:

```text
Requirements
    |
    v
Entities and Aggregates
    |
    v
Keys and Relationships
    |
    v
Access Patterns
    |
    v
Indexes and Query Models
    |
    v
Normalization or Denormalization
    |
    v
Consistency and Transactions
    |
    v
Replication and Scaling
    |
    v
Partitioning or Sharding
```

A schema is not only a storage structure.

It is a design optimized for:

- Correctness
- Query efficiency
- Write efficiency
- Evolvability
- Operational safety
- Security
- Recoverability

---

# 4. Entities, Aggregates, Keys, and Relationships

## 4.1 Entity

An entity has a stable identity.

Examples:

- Customer
- Order
- Product
- Payment
- Conversation
- Message

Questions:

- What is its identity?
- Is it mutable?
- Who owns it?
- What is its lifecycle?
- Does it have independent meaning?
- Does it contain sensitive data?

## 4.2 Value Object

A value object is defined by its attributes rather than identity.

Examples:

- Address
- Money
- Date range
- Geographic coordinate

Value objects are often embedded within aggregates.

## 4.3 Aggregate

An aggregate is a consistency boundary.

Example:

```text
Order
  |
  +-- OrderItem
  +-- ShippingAddress
  +-- Totals
```

The Order is the aggregate root.

A transaction often modifies one aggregate at a time.

This concept matters for:

- Document boundaries
- Microservices
- Event-driven systems
- Distributed transactions
- Domain ownership

## 4.4 Primary Key

A primary key uniquely identifies a record.

Qualities:

- Unique
- Stable
- Non-null
- Efficient to index
- Suitable for distribution

## 4.5 Natural Key

A natural key comes from the business domain.

Examples:

- Email
- Product SKU
- Country code
- Tax identifier

Advantages:

- Business meaning
- Natural uniqueness

Limitations:

- May change
- May be sensitive
- May be large
- May have complex uniqueness rules

## 4.6 Surrogate Key

A generated identifier.

Examples:

- Auto-increment integer
- UUID
- ULID
- Snowflake-style ID

Advantages:

- Stable
- Decoupled from business fields
- Easy to reference

## 4.7 Auto-Increment vs UUID

| Topic | Auto Increment | UUID |
|---|---|---|
| Size | Small | Larger |
| Ordering | Naturally ordered | Random unless time-based |
| Distributed generation | Harder | Easy |
| Index locality | Good | Random UUIDs may fragment indexes |
| Guessability | Predictable | Harder to guess |

Time-sortable IDs can combine distributed generation with improved index locality.

## 4.8 Foreign Keys

Foreign keys enforce relational integrity.

Advantages:

- Prevent orphaned records
- Enforce valid references
- Centralize integrity rules

Trade-offs:

- Cross-shard foreign keys are difficult
- Very high write throughput may make constraints expensive
- Some distributed stores do not support them

## 4.9 Relationship Types

### One-to-One

```text
User 1 ------ 1 UserProfile
```

### One-to-Many

```text
Customer 1 ------ N Order
```

### Many-to-Many

```text
User N ------ N Role
```

Relational implementation:

```text
User
Role
UserRole
```

Document systems may embed or reference depending on cardinality and update behavior.

---

# 5. Relational Databases

Relational databases store structured data in tables and enforce relationships through keys and constraints.

Examples:

- PostgreSQL
- MySQL
- Oracle
- SQL Server
- DB2

## 5.1 Core Strengths

- ACID transactions
- Joins
- Constraints
- Mature SQL
- Strong consistency
- Rich indexing
- Ad hoc queries
- Mature operational tooling

## 5.2 Best-Fit Workloads

- Payments
- Order management
- Banking
- Inventory
- Subscription billing
- Enterprise workflows
- Systems with complex relationships
- Systems requiring multi-row transactions

## 5.3 Example Schema

```sql
CREATE TABLE customer (
    customer_id UUID PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    name VARCHAR(200) NOT NULL,
    created_at TIMESTAMP NOT NULL
);

CREATE TABLE orders (
    order_id UUID PRIMARY KEY,
    customer_id UUID NOT NULL,
    status VARCHAR(30) NOT NULL,
    total_amount DECIMAL(12, 2) NOT NULL,
    created_at TIMESTAMP NOT NULL,
    CONSTRAINT fk_order_customer
        FOREIGN KEY (customer_id)
        REFERENCES customer(customer_id)
);
```

## 5.4 Constraints

Examples:

- Primary key
- Foreign key
- Unique
- Not null
- Check constraint

Constraints protect data even if multiple applications write to the database.

## 5.5 Joins

```sql
SELECT o.order_id, c.name
FROM orders o
JOIN customer c
  ON c.customer_id = o.customer_id
WHERE o.status = 'PROCESSING';
```

Joins are powerful, but very large distributed joins can become expensive.

## 5.6 Transactions

Relational databases are strong when multiple related changes must commit atomically.

Example:

```text
Create payment
Update invoice
Add ledger entry
```

## 5.7 Limitations

- Schema migrations require coordination
- Cross-region writes are difficult
- Horizontal write scaling may require sharding
- Complex joins can become expensive at large scale
- Flexible sparse data may not map cleanly

## 5.8 Senior Interview Point

Relational does not mean "cannot scale."

Relational systems scale through:

- Indexing
- Query optimization
- Read replicas
- Partitioning
- Caching
- Connection pooling
- Sharding
- Distributed relational products

The real question is whether the complexity is justified.

---

# 6. Document Databases

Document databases store JSON-like documents.

Examples:

- MongoDB
- Couchbase
- Amazon DocumentDB

## 6.1 Aggregate-Oriented Modeling

A document often represents one aggregate.

```json
{
  "orderId": "order-1001",
  "customerId": "cust-101",
  "status": "CREATED",
  "shippingAddress": {
    "city": "Chicago",
    "state": "IL"
  },
  "items": [
    {
      "productId": "prod-501",
      "name": "Keyboard",
      "quantity": 2,
      "unitPrice": 25.00
    }
  ]
}
```

This is efficient when the complete order is usually read together.

## 6.2 Embedding

Embed when:

- Data belongs to the same aggregate
- Data is read together
- Child cardinality is bounded
- Child data has no independent lifecycle
- Atomic document updates are valuable

Examples:

- Order items in an order
- Addresses in a user profile
- Product attributes in a product document

## 6.3 Referencing

Reference when:

- Data is shared
- Data changes independently
- Collections can grow without limit
- Independent access is common
- Duplication would be expensive

Examples:

- Product reference from an order
- User reference from conversations
- Shared category reference

## 6.4 Flexible Schema Does Not Mean No Schema

Production document systems still require:

- Validation
- Naming standards
- Versioning
- Migration strategy
- Required-field rules
- Index governance

"Schema-less" generally means schema is enforced less centrally, not that schema design is unnecessary.

## 6.5 Document Growth

Unbounded arrays are dangerous.

```json
{
  "userId": "user-101",
  "allEvents": [
    "... potentially millions ..."
  ]
}
```

Problems:

- Large reads
- Expensive updates
- Size limits
- Lock contention
- Poor pagination

Use separate collections for unbounded relationships.

## 6.6 Transactions

Some document databases support multi-document transactions.

However, frequent cross-document transactions may indicate that aggregate boundaries are wrong or that a relational model fits better.

## 6.7 Best-Fit Workloads

- Product catalogs
- Content management
- User profiles
- Flexible metadata
- Aggregate-centric applications
- Rapidly evolving fields

## 6.8 Limitations

- Duplication
- Application-managed integrity
- Expensive cross-document relationships
- Large-document risks
- Secondary indexes add cost
- Ad hoc relational queries may be harder

---

# 7. Key-Value Stores

A key-value store maps a key to an opaque or semi-structured value.

Examples:

- Redis
- Aerospike
- Riak
- DynamoDB used as direct key-value access

## 7.1 Core Access Pattern

```text
GET key
PUT key value
DELETE key
```

Its strength comes from predictable key-based access.

## 7.2 Key Design

```text
session:user-101
cart:user-101
feature:checkout-v2
idempotency:payment-request-123
rate-limit:user-101:2026-07-25T15
```

A key strategy should consider:

- Namespace
- Entity identity
- Version
- Tenant identity
- Time bucket
- Collision prevention

## 7.3 Best-Fit Workloads

- Cache
- Sessions
- Counters
- Rate limits
- Feature flags
- Shopping carts
- Idempotency records
- Leaderboards
- Simple lookup services

## 7.4 TTL

TTL is useful for:

- Sessions
- Temporary locks
- Rate limits
- Idempotency records
- One-time tokens

## 7.5 Atomic Operations

Some key-value systems support atomic:

- Increment
- Compare-and-set
- Set-if-absent
- List operations
- Sorted sets

These enable:

- Counters
- Distributed locks
- Rate limiting
- Leaderboards

## 7.6 Limitations

- Limited ad hoc queries
- Relationships handled externally
- Secondary indexes may be absent
- Scanning all keys is undesirable
- Persistence and consistency vary by product

## 7.7 Senior Interview Point

A key-value store is excellent for exact-key lookup.

It is a poor choice when users require arbitrary filtering across many fields.

---

# 8. Wide-Column Databases

Wide-column databases distribute rows by partition key and often order them by clustering columns.

Examples:

- Apache Cassandra
- ScyllaDB
- HBase
- Google Bigtable

## 8.1 Query-First Modeling

Relational approach:

```text
Model entities, then query them.
```

Wide-column approach:

```text
List queries, then create tables for those queries.
```

## 8.2 Example

Requirement:

```text
Get the latest orders for a customer.
```

Possible primary key:

```text
PRIMARY KEY ((customer_id), created_at, order_id)
```

Where:

- `customer_id` is the partition key
- `created_at` and `order_id` are clustering columns

Rows for one customer are colocated and ordered.

## 8.3 Denormalized Tables

Another query:

```text
Get orders by status and day.
```

Create another query-specific table:

```text
orders_by_status_day
```

The same order may appear in multiple tables.

## 8.4 Best-Fit Workloads

- Time-series data
- Event history
- IoT telemetry
- Activity feeds
- High-volume writes
- Global distributed data
- Query-by-partition-key workloads

## 8.5 Partition Key

A partition key determines which node stores data.

A good key:

- Distributes load
- Avoids large partitions
- Matches queries
- Avoids global scans

## 8.6 Clustering Key

A clustering key controls ordering within a partition.

Useful for:

- Time ranges
- Latest N records
- Ordered event history

## 8.7 Tombstones

Deletes and expirations may create tombstones.

Too many tombstones increase read cost.

Consider:

- TTL usage
- Compaction
- Deletion volume
- Partition size

## 8.8 Tunable Consistency

Some systems allow consistency levels conceptually such as:

- One replica
- Quorum
- All replicas

Higher consistency may increase latency and reduce availability.

## 8.9 Limitations

- Joins avoided
- Cross-partition queries expensive
- Query changes may need new tables
- Denormalized writes
- Operational complexity
- Hot partitions
- Large partitions
- Limited ad hoc analytics

---

# 9. Graph Databases

Graph databases represent data using:

- Nodes
- Edges
- Properties

Examples:

- Neo4j
- Amazon Neptune
- JanusGraph

## 9.1 Example

```text
(User:101)-[:FOLLOWS]->(User:202)

(User:101)-[:PURCHASED]->(Product:501)

(Product:501)-[:BELONGS_TO]->(Category:Electronics)
```

## 9.2 Best-Fit Workloads

- Social relationships
- Fraud networks
- Recommendations
- Dependency analysis
- Network topology
- Identity relationships
- Knowledge graphs

## 9.3 Traversal

Graph databases excel when queries follow relationships.

Example:

```text
Find friends of friends who purchased the same product.
```

A relational database may require repeated self-joins.

A graph database follows edges naturally.

## 9.4 Property Graph

```text
(User {id:101})
  -[:PURCHASED {date:"2026-07-25"}]->
(Product {id:501})
```

## 9.5 Index-Free Adjacency

Some graph databases store direct references between connected nodes.

Traversal cost depends more on the visited subgraph than the full dataset.

## 9.6 Limitations

- Not ideal for simple CRUD at all scales
- Horizontal scaling may be difficult
- Specialized query languages
- Limited team expertise
- Often complements another operational database
- Bulk aggregation may be better elsewhere

# 10. Access-Pattern-Driven Design

Schema design must start with the most important access patterns.

## 10.1 Read Patterns

Examples:

- Get by ID
- Get latest N records by owner
- Search by multiple fields
- Scan by time range
- Aggregate by region
- Traverse relationships
- Retrieve a complete aggregate

## 10.2 Write Patterns

Examples:

- Insert-only events
- Frequent partial updates
- Large batch writes
- Counter increments
- Multi-record transactions
- Conditional updates
- Append-only logs

## 10.3 Read-Heavy vs Write-Heavy

Read-heavy systems may use:

- Caching
- Read replicas
- Denormalized views
- Search indexes
- Materialized views

Write-heavy systems may use:

- Append-oriented storage
- Fewer indexes
- Partitioned writes
- Batch ingestion
- Asynchronous materialization

## 10.4 Query Matrix

| Query | Frequency | Latency Target | Consistency | Result Size |
|---|---:|---:|---|---:|
| Get order by ID | High | <100 ms | Strong | 1 |
| List customer orders | High | <200 ms | Read-your-writes | 50 |
| Search products | High | <300 ms | Eventual | 20 |
| Daily sales report | Low | Minutes | Eventual | Large |

This matrix helps determine:

- Source of truth
- Indexes
- Cache
- Search engine
- Read replica
- Batch analytics
- Denormalized read model

## 10.5 Query-First Does Not Mean Query-Only

The design must also protect:

- Data integrity
- Maintainability
- Schema evolution
- Write amplification
- Operational cost

Optimizing every query with a duplicate table can create an unmanageable system.

---

# 11. Indexing

Indexes are additional data structures that accelerate reads.

They trade:

```text
More storage and write cost
for
Faster reads
```

## 11.1 B-Tree Index

Good for:

- Equality
- Range
- Sorting
- Prefix matching on indexed columns

## 11.2 Hash Index

Good for:

- Equality lookup

Poor for:

- Range queries
- Ordering

## 11.3 Composite Index

Example query:

```sql
SELECT *
FROM orders
WHERE customer_id = ?
  AND status = ?
ORDER BY created_at DESC;
```

Possible index:

```text
(customer_id, status, created_at DESC)
```

## 11.4 Leftmost Prefix

For an index:

```text
(customer_id, status, created_at)
```

It commonly supports:

```text
customer_id
customer_id + status
customer_id + status + created_at
```

It may not efficiently support:

```text
status only
created_at only
```

Exact optimizer behavior varies by database.

## 11.5 Covering Index

A covering index contains all fields required by a query.

This can avoid reading the base table.

Trade-off:

- Larger index
- More write cost
- More storage

## 11.6 Selectivity

A highly selective column narrows results significantly.

Examples:

- Order ID: high selectivity
- Boolean active flag: low selectivity

Low-selectivity fields may be useful only when combined with other columns.

## 11.7 Index and Sort Order

Index order should consider:

- Equality filters
- Range filters
- Sort requirements
- Cardinality
- Query frequency

This is a design heuristic, not a substitute for execution-plan analysis.

## 11.8 Partial or Filtered Index

A partial index covers only selected rows.

Example:

```text
Index only active orders.
```

This can reduce index size and write cost.

## 11.9 Functional Index

An index may be built on an expression.

Example:

```text
lower(email)
```

Useful for case-insensitive lookup.

## 11.10 Full-Text Index

Use a dedicated full-text index for:

- Tokenization
- Relevance
- Stemming
- Fuzzy matching

A normal B-tree index is not a replacement for full-text search.

## 11.11 Over-Indexing

Too many indexes cause:

- Slower inserts
- Slower updates
- Larger storage
- More memory pressure
- More replication traffic
- Longer migrations

## 11.12 Query Plans

Senior engineers should inspect execution plans.

Look for:

- Full scans
- Index scans
- Row estimates
- Join order
- Sort operations
- Temporary storage
- Actual vs estimated rows
- Partition pruning

## 11.13 Distributed Secondary Indexes

Secondary indexes in distributed databases may:

- Create additional partitions
- Increase write amplification
- Require coordination
- Cause scatter-gather queries
- Have limited consistency guarantees

Use them intentionally.

---

# 12. Normalization

Normalization reduces duplication and update anomalies.

## 12.1 First Normal Form

Values are atomic and rows are identifiable.

Avoid:

```text
phone_numbers = "111,222,333"
```

Prefer a related table.

## 12.2 Second Normal Form

Non-key attributes depend on the full key.

This matters especially for composite keys.

## 12.3 Third Normal Form

Non-key attributes depend on the key, not on other non-key attributes.

Example of transitive dependency:

```text
employee_id -> department_id -> department_name
```

Store department data separately.

## 12.4 Benefits

- Consistent updates
- Less duplication
- Strong integrity
- Smaller writes
- Clear ownership
- Easier business-rule enforcement

## 12.5 Costs

- More joins
- More tables
- More query complexity
- Potentially higher read latency
- Harder distributed access

Normalization is particularly valuable for transactional source-of-truth data.

---

# 13. Denormalization

Denormalization duplicates or precomputes data to improve reads.

## 13.1 Examples

- Store customer name on an order snapshot
- Store product summary in a search index
- Maintain daily sales totals
- Store follower count on a user profile
- Create an orders-by-customer table
- Store prejoined dashboard data

## 13.2 Valid Reasons

- Avoid expensive joins
- Support partition-local reads
- Precompute aggregates
- Serve low-latency queries
- Support read models
- Reduce cross-service calls
- Preserve historical snapshots

## 13.3 Risks

- Stale data
- Duplicate storage
- Update fan-out
- Repair complexity
- Eventual consistency
- More complicated migrations
- Higher write amplification

## 13.4 Snapshot vs Synchronization

Sometimes duplicated data is intentionally historical.

Example:

```text
Order stores product name and price at purchase time.
```

This is a transaction-time snapshot, not necessarily data that should track the current product record.

## 13.5 Maintaining Denormalized Data

Methods:

- Synchronous transaction
- Domain event
- Change data capture
- Scheduled reconciliation
- Materialized view
- Batch rebuild

Choose based on:

- Freshness
- Reliability
- Write latency
- Failure recovery
- Operational complexity

---

# 14. Transactions and Consistency

## 14.1 ACID

### Atomicity

All operations succeed or all fail.

### Consistency

The transaction preserves database rules and invariants.

### Isolation

Concurrent transactions behave safely.

### Durability

Committed data survives failure.

## 14.2 Isolation Levels

Common conceptual levels:

- Read uncommitted
- Read committed
- Repeatable read
- Serializable
- Snapshot isolation

Higher isolation reduces anomalies but may lower concurrency.

## 14.3 Common Anomalies

- Dirty read
- Non-repeatable read
- Phantom read
- Lost update
- Write skew

## 14.4 Optimistic Locking

Use a version field.

```sql
UPDATE orders
SET status = ?, version = version + 1
WHERE order_id = ?
  AND version = ?;
```

Best when conflicts are uncommon.

## 14.5 Pessimistic Locking

Lock data before modification.

Best when:

- Conflicts are frequent
- Critical invariants must be protected

Risks:

- Blocking
- Deadlocks
- Lower throughput
- Long-running locks

## 14.6 Conditional Writes

Distributed databases often use conditional updates or compare-and-set.

Example:

```text
Update status only if current status is CREATED.
```

This protects state transitions without a long lock.

## 14.7 Distributed Transactions

Across services or databases, one ACID transaction may not be available.

Patterns:

- Saga
- Transactional outbox
- Idempotency
- Compensation
- Reconciliation

## 14.8 Strong vs Eventual Consistency

Strong consistency fits:

- Payments
- Balances
- Inventory reservation
- Authorization
- Unique business constraints

Eventual consistency often fits:

- Search indexes
- Analytics
- Feeds
- Recommendations
- Counters
- Reporting

## 14.9 Read-Your-Writes

A user may need to see their recent update even when replicas lag.

Techniques:

- Read from leader after write
- Session stickiness
- Version token
- Cache update
- Monotonic-read routing

---

# 15. Replication

Replication creates copies of data.

## 15.1 Leader-Follower

```text
Leader
  |
  +----> Follower 1
  |
  +----> Follower 2
```

The leader handles writes.

Followers may serve reads.

## 15.2 Benefits

- Read scaling
- High availability
- Disaster recovery
- Geographic locality

## 15.3 Replication Lag

A write may not immediately appear on followers.

Consequences:

- Stale reads
- Missing recently created records
- Non-monotonic reads
- User cannot see own update

Solutions:

- Read from leader temporarily
- Session stickiness
- Version token
- Monotonic-read routing
- Cache update

## 15.4 Synchronous Replication

Leader waits for replicas.

Advantages:

- Stronger durability
- Lower data-loss risk

Costs:

- Higher latency
- Lower availability if replicas are unavailable

## 15.5 Asynchronous Replication

Leader commits before replicas acknowledge.

Advantages:

- Lower latency
- Better write availability

Costs:

- Replication lag
- Possible recent data loss during failover

## 15.6 Multi-Leader

Multiple leaders accept writes.

Advantages:

- Regional write locality
- Better write availability

Challenges:

- Conflict resolution
- Duplicate records
- Clock issues
- Complex failover
- Consistency semantics

## 15.7 Leaderless Replication

Some distributed stores allow reads and writes to multiple replicas.

Concepts may include:

- Replication factor
- Read quorum
- Write quorum
- Read repair
- Anti-entropy

The trade-off is tunable consistency versus latency and availability.

---

# 16. Partitioning and Sharding

Partitioning divides data into subsets.

Sharding commonly means distributing those subsets across nodes.

## 16.1 Horizontal Partitioning

Rows are divided.

```text
Shard 1: users A-F
Shard 2: users G-M
Shard 3: users N-Z
```

## 16.2 Vertical Partitioning

Columns or functional areas are separated.

Example:

```text
User core data
User preferences
User audit history
```

## 16.3 Range Sharding

```text
Customer IDs 1-1,000,000 -> Shard 1
Customer IDs 1,000,001-2,000,000 -> Shard 2
```

Advantages:

- Natural range queries
- Easy ordered scans

Risks:

- Uneven growth
- Hot newest range
- Manual split management

## 16.4 Hash Sharding

```text
hash(customer_id) % shard_count
```

Advantages:

- More even distribution

Limitations:

- Range queries are harder
- Resharding can move much data

## 16.5 Directory-Based Sharding

A lookup service maps records to shards.

Advantages:

- Flexible placement
- Supports special routing

Limitations:

- Additional dependency
- Lookup overhead
- Operational complexity

## 16.6 Consistent Hashing

Consistent hashing reduces key movement when nodes join or leave.

Common in:

- Distributed caches
- Storage systems
- Partitioned services

## 16.7 Geographic Sharding

Partition by region.

Advantages:

- Data locality
- Residency compliance
- Lower user latency

Challenges:

- User movement
- Cross-region queries
- Global uniqueness
- Disaster recovery
- Cross-region transactions

## 16.8 Tenant Sharding

For SaaS systems, tenant ID may be a natural shard key.

Risks:

- One very large tenant
- Uneven tenant sizes
- Noisy-neighbor behavior

A hybrid strategy may place large tenants on dedicated shards.

## 16.9 Cross-Shard Operations

Cross-shard joins and transactions are expensive.

Reduce them through:

- Good shard key
- Local denormalized data
- Asynchronous aggregation
- Global lookup service
- Batch analytics
- Saga patterns

## 16.10 Resharding

Resharding may require:

- Data copy
- Dual reads
- Dual writes
- Backfill
- Routing update
- Validation
- Cutover
- Cleanup

Plan before a single shard becomes too large.

---

# 17. Hot Partitions and Skew

A hot partition receives disproportionate traffic.

Causes:

- Celebrity account
- Current timestamp key
- One large tenant
- Sequential writes
- Popular product
- Low-cardinality partition key

## 17.1 Detection

Monitor:

- Requests per partition
- Storage per partition
- CPU per node
- Throttling
- Latency
- Error rate
- Queue depth

## 17.2 Mitigation

- Hash or salt the key
- Split a large tenant
- Add time buckets
- Replicate hot read keys
- Cache hot data
- Use load-aware routing
- Precompute results
- Separate high-volume tenants

## 17.3 Time Bucketing

Instead of:

```text
partition_key = device_id
```

Use:

```text
partition_key = device_id + day
```

This limits partition size.

Trade-off:

- Multi-day queries touch multiple partitions

## 17.4 Write Sharding

For one hot logical key, append a bucket:

```text
counter:product-501:bucket-0
counter:product-501:bucket-1
counter:product-501:bucket-2
```

Read by aggregating buckets.

This improves write distribution at the cost of read complexity.

# 18. Schema Evolution and Migration

Production schemas change continuously.

## 18.1 Backward-Compatible Changes

Prefer:

- Add nullable column
- Add a field with safe default behavior
- Add a new table or collection
- Add a new index online where supported
- Add a new event field
- Keep the old field during migration

## 18.2 Expand-and-Contract Pattern

### Expand

Add the new schema while old code still works.

### Migrate

Backfill data and switch reads and writes gradually.

### Contract

Remove the old schema after all consumers migrate.

```text
Old Schema
    |
    v
Old + New Schema
    |
    v
Backfill and Dual Compatibility
    |
    v
New Schema
    |
    v
Remove Old Schema
```

## 18.3 Dual Read

Read from the new store first and fall back to the old store.

Benefits:

- Gradual migration
- Safer cutover

Risks:

- Hidden inconsistency
- Increased read complexity
- Longer migration period

## 18.4 Dual Write

Write to old and new stores during migration.

Risks:

- One write succeeds and the other fails
- Ordering differences
- Retry duplicates
- Difficult rollback

Use:

- Idempotency
- Reconciliation
- Durable event log
- Clear source of truth

## 18.5 Backfill

A backfill should be:

- Rate limited
- Restartable
- Observable
- Idempotent
- Partitioned
- Validated

Monitor:

- Rows completed
- Error count
- Throughput
- Replication lag
- Database load
- Mismatch count

## 18.6 Zero-Downtime Index Creation

Large index creation may:

- Lock writes
- Consume CPU
- Increase disk usage
- Increase replication lag

Use online or concurrent creation when available, and monitor production impact.

## 18.7 Event and Document Versioning

Events and documents should include a version where evolution matters.

Consumers should tolerate:

- Additional fields
- Missing optional fields
- Old versions during transition

Do not silently change field meaning.

## 18.8 Database Migration Strategy

A common migration flow:

```text
Create target schema
      |
      v
Start change capture
      |
      v
Backfill historical data
      |
      v
Validate counts and checksums
      |
      v
Enable shadow reads
      |
      v
Gradually switch traffic
      |
      v
Keep rollback window
      |
      v
Retire old store
```

---

# 19. Polyglot Persistence

Polyglot persistence uses different database models for different workloads.

Example:

```text
PostgreSQL:
Orders and payments

Redis:
Sessions, cache, rate limits, idempotency

OpenSearch:
Product search

Cassandra:
Activity history and telemetry

Graph Database:
Fraud relationship analysis

Object Storage:
Images and documents
```

## 19.1 Advantages

- Best fit for each workload
- Independent scaling
- Specialized capabilities
- Better performance for critical access patterns

## 19.2 Costs

- More operational systems
- More skills required
- Data synchronization
- More failure modes
- More monitoring
- Higher cost
- More complex disaster recovery
- Harder governance

## 19.3 Senior Decision Rule

Use another database only when the specialized benefit clearly exceeds the operational cost.

Do not introduce a graph database for one simple relationship query or a wide-column store for a workload that fits a relational database comfortably.

---

# 20. Security and Governance

## 20.1 Data Classification

Classify data as:

- Public
- Internal
- Confidential
- Restricted

Classification determines:

- Encryption
- Access controls
- Retention
- Audit requirements
- Masking
- Backup policy

## 20.2 Encryption

Use encryption:

- In transit
- At rest
- For backups
- For snapshots
- For replication channels

## 20.3 Least Privilege

Applications should access only required:

- Databases
- Tables
- Collections
- Key prefixes
- Stored procedures
- Operations

Separate:

- Read-only users
- Read-write users
- Migration users
- Administrative users

## 20.4 Sensitive Data

Avoid storing unnecessary:

- Passwords
- Tokens
- Payment data
- Personal identifiers
- Secrets

Use:

- Secure password hashing
- Tokenization
- Masking
- Field-level encryption
- Data minimization

## 20.5 Audit

Audit:

- Schema changes
- Permission changes
- Sensitive reads
- Administrative writes
- Data exports
- Deletion requests

## 20.6 Multi-Tenancy Models

### Shared Table

```text
tenant_id included in every table
```

Advantages:

- Cost efficient
- Easy to onboard tenants

Risks:

- Data-leak bugs
- Noisy neighbors
- Large shared indexes

### Separate Schema

Advantages:

- Better logical isolation
- Easier tenant-specific migration

Risks:

- Many schemas to manage

### Separate Database

Advantages:

- Strong isolation
- Tenant-specific scaling
- Easier compliance boundaries

Risks:

- Higher cost
- Operational overhead
- Connection management complexity

## 20.7 Retention and Deletion

Design for:

- Legal retention
- User deletion
- Archiving
- Backup retention
- Event expiration
- Data anonymization

Deleting from the primary database is not enough if data remains in:

- Cache
- Search index
- Analytics
- Backups
- Event streams
- Replicas

---

# 21. Observability and Operations

Monitor:

- Query latency
- P95 and P99 latency
- Slow queries
- Lock waits
- Deadlocks
- Connection pool usage
- Cache hit ratio
- Replica lag
- Disk usage
- Index size
- Compaction
- Tombstones
- Partition skew
- Throttling
- Backup success
- Restore testing

## 21.1 Connection Pooling

Too few connections:

- Requests wait
- Throughput drops

Too many connections:

- Database overload
- Memory pressure
- Context switching
- Lock contention

Pool sizing should consider:

- Database capacity
- Number of service instances
- Query latency
- Request concurrency
- Connection lifetime

## 21.2 Slow Query Process

```text
Identify slow query
      |
      v
Inspect execution plan
      |
      v
Check indexes and row estimates
      |
      v
Reduce scanned data or joins
      |
      v
Measure again
```

## 21.3 Backup Is Not Enough

A backup strategy requires:

- Retention
- Encryption
- Restore testing
- RPO
- RTO
- Regional copy
- Point-in-time recovery
- Ownership and runbooks

## 21.4 Capacity Monitoring

Track:

- Storage growth
- Index growth
- Partition size
- Write amplification
- Read/write throughput
- CPU and memory
- Network throughput
- Cache eviction
- Compaction backlog

## 21.5 Database Failure Scenarios

Prepare for:

- Primary failure
- Replica failure
- Region failure
- Corrupted deployment
- Accidental deletion
- Bad schema migration
- Credential compromise
- Storage exhaustion
- Network partition

---

# 22. Scenario-Based Selection

## 22.1 Payment Ledger

Choose a relational database.

Reasons:

- Strong transactions
- Constraints
- Auditability
- Ordered financial records
- Reconciliation
- Precise numeric types

A separate immutable ledger may record debit and credit entries.

## 22.2 Product Catalog

A document database can fit when:

- Categories have different attributes
- Product data is read as one aggregate
- Fields evolve frequently

Add a search engine for:

- Full-text search
- Facets
- Relevance
- Autocomplete

## 22.3 Session Store

Choose key-value.

Requirements:

- Direct lookup
- TTL
- High throughput
- Simple access
- Fast expiration

## 22.4 Idempotency Store

A key-value or relational store can work.

The record may include:

```text
Idempotency Key
Request Hash
Processing Status
Response
Expiration
```

Choice depends on:

- Durability
- Transaction relationship
- Throughput
- Retention

## 22.5 IoT Telemetry

Choose a wide-column or time-series-oriented store.

Requirements:

- Massive writes
- Time-range queries
- Partition by device and time
- TTL and retention
- Append-heavy workload

## 22.6 Fraud Detection

Use a graph database for relationship traversal.

Transactional source data may remain in a relational database.

## 22.7 Social Feed

Possible architecture:

- Relational source of truth for users and posts
- Wide-column or key-value feed store
- Cache for hot feeds
- Search index for discovery
- Asynchronous fan-out

## 22.8 Order Management

Relational is a common source of truth.

Additional stores may include:

- Redis cache
- Search index
- Message broker
- Reporting warehouse

## 22.9 Shopping Cart

A key-value or document model may fit because:

- Access is normally by user
- Cart is an aggregate
- Low latency matters
- Expiration may be required

Persist important checkout state to a durable transactional store.

## 22.10 Recommendation System

Possible combination:

- Graph database for relationships
- Key-value store for precomputed recommendations
- Object or analytical store for training data
- Relational store for source entities

## 22.11 Audit Log

An append-oriented relational, wide-column, or event store may fit.

Requirements:

- Immutable records
- Time-based access
- Retention
- Tamper evidence
- Search or archive

## 22.12 Multi-Tenant SaaS

Choice depends on:

- Tenant count
- Largest tenant
- Isolation
- Compliance
- Query patterns

A common progression:

```text
Shared database with tenant_id
        |
        v
Partition by tenant
        |
        v
Move very large tenants to dedicated shards
```

---

# 23. Trade-Off Matrix

| Dimension | Relational | Document | Key-Value | Wide-Column | Graph |
|---|---|---|---|---|---|
| Transactions | Strong | Often aggregate-focused; product-dependent | Limited or product-dependent | Limited or tunable | Product-dependent |
| Joins | Strong | Limited or application-managed | No | Avoided | Traversal-oriented |
| Flexible schema | Moderate | Strong | Strong | Moderate | Strong |
| Direct key lookup | Strong | Strong | Excellent | Strong | Moderate |
| Range queries | Strong with indexes | Strong with indexes | Limited | Excellent within partition | Traversal-oriented |
| Relationship traversal | Good with joins | Limited | Poor | Poor | Excellent |
| Horizontal write scaling | Moderate to complex | Good in many products | Excellent | Excellent | Product-dependent |
| Ad hoc queries | Excellent | Good | Poor | Poor | Good for graph patterns |
| Source-of-truth transactions | Excellent | Good for aggregates | Narrow use | Specialized | Specialized |
| Operational simplicity | Mature | Moderate | Often simple for narrow use | Complex | Specialized |
| Best fit | Transactions and relations | Aggregates | Fast lookup | Massive partitioned workloads | Connected data |

---

# 24. Common Interview Questions

## Fundamentals

- How do you choose a database?
- SQL vs NoSQL?
- What does schema-less mean?
- What is an aggregate?
- What is an access pattern?
- Why should schema design begin with queries?
- What is polyglot persistence?

## Relational

- What are primary and foreign keys?
- Natural key vs surrogate key?
- What are normalization forms?
- What are transaction isolation levels?
- What is optimistic locking?
- What is pessimistic locking?
- What is a covering index?
- How do composite indexes work?
- Why can too many indexes be harmful?
- How do you scale a relational database?
- When would you shard PostgreSQL?

## Document

- Embed vs reference?
- How do you prevent unbounded document growth?
- How do you enforce schema?
- When are multi-document transactions a warning sign?
- How do you model many-to-many relationships?
- How do you update duplicated embedded data?

## Key-Value

- What makes a good key?
- How do TTLs work?
- How do you implement idempotency?
- How do you implement rate limiting?
- When is a key-value store a poor choice?
- How do you handle hot keys?

## Wide-Column

- What is query-first modeling?
- What is a partition key?
- What is a clustering key?
- Why are joins avoided?
- What creates hot partitions?
- What are tombstones?
- How do tunable consistency levels work?
- Why might one entity be written to multiple tables?

## Graph

- When is a graph database better than relational?
- What are nodes and edges?
- What is multi-hop traversal?
- What is index-free adjacency?
- Why might graph be used alongside relational?

## Indexing

- What is index selectivity?
- What is the leftmost-prefix rule?
- Equality vs range ordering in a composite index?
- What is a partial index?
- What is a functional index?
- How do you analyze a query plan?
- Why might the optimizer ignore an index?

## Scaling

- Partitioning vs sharding?
- Range vs hash sharding?
- What is consistent hashing?
- What is replication lag?
- Synchronous vs asynchronous replication?
- How do you handle cross-shard queries?
- How do you reshard?
- What is a hot partition?
- How do you shard a large tenant?

## Migration

- How do you perform zero-downtime schema changes?
- What is expand-and-contract?
- How do you backfill safely?
- How do you migrate between databases?
- How do you validate dual writes?
- How do you roll back a database migration?

## Senior Scenario Questions

- Design storage for an order system.
- Design a product catalog with different category attributes.
- Design a social feed database.
- Design a payment ledger.
- Design storage for billions of device events.
- Design a multi-tenant database strategy.
- Explain when you would introduce a search engine.
- Explain how you would reduce database latency by 60%.
- Explain how you would handle a hot customer or tenant.
- Explain how you would choose consistency for each data type.

---

# 25. Interview Readiness Tracker

## Requirements and Modeling

- [ ] I start with requirements.
- [ ] I list critical queries.
- [ ] I identify read and write patterns.
- [ ] I identify entities and aggregates.
- [ ] I choose stable keys.
- [ ] I explain relationships.
- [ ] I identify consistency boundaries.
- [ ] I estimate growth and retention.

## Relational

- [ ] I understand ACID.
- [ ] I understand isolation levels.
- [ ] I understand normalization.
- [ ] I understand constraints.
- [ ] I can design composite indexes.
- [ ] I can read a query plan at a high level.
- [ ] I understand optimistic and pessimistic locking.
- [ ] I can explain relational scaling.

## Document

- [ ] I explain aggregate-oriented storage.
- [ ] I choose embedding vs referencing.
- [ ] I understand document growth.
- [ ] I understand flexible-schema governance.
- [ ] I understand document transaction boundaries.
- [ ] I can maintain duplicated fields.

## Key-Value

- [ ] I can design keys.
- [ ] I understand TTL.
- [ ] I understand atomic operations.
- [ ] I can explain sessions, rate limits, counters, and idempotency.
- [ ] I know when key-value is insufficient.
- [ ] I can address hot keys.

## Wide-Column

- [ ] I understand query-first modeling.
- [ ] I can choose partition and clustering keys.
- [ ] I understand denormalized query tables.
- [ ] I understand tombstones and compaction.
- [ ] I can explain hot partitions.
- [ ] I understand tunable consistency.
- [ ] I understand partition-size limits.

## Graph

- [ ] I understand nodes, edges, and properties.
- [ ] I can identify graph workloads.
- [ ] I compare traversal with relational joins.
- [ ] I understand graph operational trade-offs.
- [ ] I know when graph should complement another store.

## Indexing

- [ ] I understand B-tree and hash indexes.
- [ ] I understand composite index order.
- [ ] I understand selectivity.
- [ ] I understand covering indexes.
- [ ] I understand partial and functional indexes.
- [ ] I understand over-indexing.
- [ ] I inspect query plans.

## Normalization and Denormalization

- [ ] I explain 1NF, 2NF, and 3NF.
- [ ] I justify denormalization.
- [ ] I explain historical snapshots.
- [ ] I maintain denormalized views safely.
- [ ] I understand eventual consistency.
- [ ] I understand write amplification.

## Replication and Sharding

- [ ] I understand leader-follower replication.
- [ ] I understand leaderless and multi-leader concepts.
- [ ] I understand replication lag.
- [ ] I compare synchronous and asynchronous replication.
- [ ] I understand range and hash sharding.
- [ ] I can choose a shard key.
- [ ] I understand cross-shard operations.
- [ ] I can explain resharding.
- [ ] I detect and mitigate hot partitions.
- [ ] I understand tenant sharding.

## Evolution and Operations

- [ ] I explain expand-and-contract.
- [ ] I can plan a safe backfill.
- [ ] I understand dual-read and dual-write risks.
- [ ] I monitor slow queries and locks.
- [ ] I monitor replica lag and partition skew.
- [ ] I understand connection-pool sizing.
- [ ] I understand backup and restore requirements.
- [ ] I explain polyglot-persistence trade-offs.
- [ ] I can create a rollback plan.

---

# 26. Senior Interview Answer

> I select a database by first understanding the business requirements, data relationships, access patterns, transaction boundaries, consistency expectations, and expected scale. Relational databases are my default when strong transactions, constraints, joins, and ad hoc querying matter. Document databases fit aggregate-oriented data with nested structures and frequently evolving fields, but I still define schema rules and carefully choose between embedding and referencing. Key-value stores are best for direct-key workloads such as caching, sessions, counters, rate limits, and idempotency. Wide-column databases are appropriate for very large distributed workloads with high write throughput and predictable partition-key queries, where query-first modeling and denormalization are acceptable. Graph databases are useful when multi-hop relationship traversal is the primary workload, such as fraud detection, identity analysis, or recommendations.

> For schema design, I begin with requirements and the highest-value read and write patterns. I identify entities, aggregates, keys, and relationships, then design indexes around real queries rather than indexing every column. I normalize source-of-truth transactional data where consistency matters and denormalize selectively for latency, partition locality, historical snapshots, and read scale. I scale progressively through query optimization, indexes, caching, read replicas, partitioning, and sharding. When sharding is required, I choose a high-cardinality key that distributes traffic evenly and minimizes cross-shard operations, while planning for hot partitions, resharding, migration, observability, reconciliation, and disaster recovery. I use polyglot persistence only when the value of specialized storage clearly exceeds its operational complexity.
