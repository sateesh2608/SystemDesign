# Database Models and Schema Design

A concise guide to the major database models and the fundamentals of production schema design.

This document covers:

- Relational databases
- Document databases
- Key-value stores
- Wide-column databases
- Graph databases
- Requirements-driven schema design
- Entities, keys, and relationships
- Indexing for access patterns
- Normalization and denormalization
- Replication, scaling, and sharding
- Senior-level interview questions

For deeper learning, see the [Detailed Database Models and Schema Design Interview Guide](DatabaseModelsAndSchemaDesign-DetailedInterviewGuide.md).

---

# 1. Start with Requirements, Not Technology

Do not begin with:

```text
We should use PostgreSQL.
```

or:

```text
We should use MongoDB because it scales.
```

Begin with:

```text
What data must be stored?

How is the data related?

What are the most important reads and writes?

What consistency and transaction guarantees are required?

What is the expected scale?

How frequently does the schema change?
```

Database choice should follow business requirements and access patterns.

---

# 2. Questions to Ask Before Selecting a Database

- What are the core entities?
- What are the relationships?
- What are the primary read queries?
- What are the primary write operations?
- Is the workload read-heavy or write-heavy?
- Are multi-record transactions required?
- Is strong consistency required?
- Can the application tolerate stale reads?
- What is the expected data size?
- What is the peak request rate?
- Is horizontal scaling required?
- Are joins important?
- Does the schema change frequently?
- Is graph traversal a primary operation?
- Are range queries required?
- Is the system multi-region?

---

# 3. Relational Database

A relational database stores data in tables containing rows and columns.

Examples:

- PostgreSQL
- MySQL
- Oracle
- SQL Server
- DB2

## Core Characteristics

- Structured schema
- Primary and foreign keys
- Relationships
- Joins
- ACID transactions
- Constraints
- SQL query language

## Example

```text
Customer
- customer_id
- name
- email

Order
- order_id
- customer_id
- status
- total_amount
```

Relationship:

```text
Customer 1 ------ N Order
```

## Best Use Cases

- Payments
- Order management
- Banking
- Inventory
- Enterprise applications
- Systems with complex relationships
- Systems requiring strong transactional integrity

## Advantages

- Strong consistency
- Transactions
- Constraints
- Joins
- Mature tooling
- Powerful query language
- Clear schema governance

## Limitations

- Horizontal write scaling can be complex
- Schema changes may require migrations
- Large distributed joins are expensive
- Highly flexible or deeply nested data can be awkward

---

# 4. Document Database

A document database stores records as JSON-like documents.

Examples:

- MongoDB
- Couchbase
- Amazon DocumentDB

## Example

```json
{
  "orderId": "order-1001",
  "customer": {
    "customerId": "cust-101",
    "name": "John"
  },
  "items": [
    {
      "productId": "prod-501",
      "quantity": 2,
      "price": 25.00
    }
  ],
  "status": "CREATED"
}
```

## Best Use Cases

- Product catalogs
- Content management
- User profiles
- Flexible metadata
- Rapidly evolving application schemas
- Aggregate-oriented data

## Advantages

- Flexible schema
- Natural mapping to application objects
- Nested data
- Fewer joins for aggregate reads
- Easier evolution for varying document shapes

## Limitations

- Data duplication
- Cross-document joins may be limited or expensive
- Large documents can be costly to update
- Referential integrity may move into application logic
- Transactions across many documents may be less natural

## Embed vs Reference

Embed when:

- Data belongs to the same aggregate
- It is usually read together
- It does not grow without limit

Reference when:

- Data is shared
- It changes independently
- It is large
- Many documents refer to it

---

# 5. Key-Value Store

A key-value store saves a value using a unique key.

Examples:

- Redis
- Aerospike
- Riak
- DynamoDB in direct key-value use cases

## Example

```text
Key:
session:user-101

Value:
{
  "userId": "user-101",
  "roles": ["CUSTOMER"],
  "expiresAt": "2026-07-25T18:00:00Z"
}
```

## Best Use Cases

- Sessions
- Caching
- Feature flags
- Counters
- Shopping carts
- Rate limiting
- Idempotency keys
- Simple high-volume lookups

## Advantages

- Very fast lookup
- Simple data model
- Easy horizontal partitioning
- High throughput
- Good for predictable key access

## Limitations

- Limited query flexibility
- Relationships are handled by the application
- Secondary access patterns can be difficult
- Poor fit for ad hoc querying
- Key design is critical

---

# 6. Wide-Column Database

A wide-column database organizes data using partition keys and clustering or sort keys.

Examples:

- Apache Cassandra
- ScyllaDB
- Google Bigtable
- HBase

## Example

```text
Partition Key: customer_id
Clustering Key: order_created_at

customer_id | order_created_at | order_id | status | total
```

Rows in one partition are stored together and ordered by the clustering key.

## Best Use Cases

- Massive write throughput
- Time-series data
- Event history
- IoT telemetry
- Activity feeds
- Large-scale logs
- Predictable partition-key queries

## Advantages

- Horizontal scalability
- High write throughput
- Multi-node availability
- Efficient range scans inside a partition
- Good for large distributed workloads

## Limitations

- Query-first schema design
- Joins are generally avoided
- Denormalization is common
- Cross-partition queries are expensive
- Poor keys create hot partitions
- Multiple query-specific tables increase write complexity

## Important Rule

Design one table around one important access pattern.

Do not model a wide-column database like a relational database.

---

# 7. Graph Database

A graph database stores nodes, relationships, and properties.

Examples:

- Neo4j
- Amazon Neptune
- JanusGraph

## Example

```text
(User:101)-[:FOLLOWS]->(User:202)

(User:101)-[:PURCHASED]->(Product:501)

(Product:501)-[:BELONGS_TO]->(Category:Electronics)
```

## Best Use Cases

- Social networks
- Fraud detection
- Recommendation engines
- Knowledge graphs
- Network topology
- Identity relationships
- Dependency analysis

## Advantages

- Fast relationship traversal
- Natural representation of connected data
- Flexible relationship modeling
- Strong for multi-hop queries

## Limitations

- Not ideal for every CRUD workload
- Operational expertise may be limited
- Horizontal scaling can be more complex
- Often complements another database rather than replacing it

---

# 8. Quick Comparison

| Database Model | Data Shape | Main Strength | Typical Use |
|---|---|---|---|
| Relational | Tables and relations | Transactions and joins | Payments, orders, enterprise systems |
| Document | JSON-like documents | Flexible aggregate storage | Catalogs, profiles, content |
| Key-value | Key to value | Fast direct lookup | Cache, sessions, counters |
| Wide-column | Partitioned rows | Massive distributed writes | Telemetry, events, feeds |
| Graph | Nodes and edges | Relationship traversal | Fraud, social graphs, recommendations |

---

# 9. Schema Design Fundamentals

Use this sequence:

```text
Requirements
    |
    v
Core Entities
    |
    v
Keys and Relationships
    |
    v
Access Patterns
    |
    v
Indexes
    |
    v
Normalization or Denormalization
    |
    v
Scaling and Sharding
```

---

# 10. Identify Core Entities

Look for nouns in requirements.

Example:

```text
A customer places an order containing products and pays using a payment method.
```

Entities:

- Customer
- Product
- Order
- Order Item
- Payment
- Payment Method

Questions:

- What uniquely identifies each entity?
- Who owns it?
- How often is it read?
- How often is it updated?
- How long is it retained?
- Does it contain sensitive data?
- What relationships exist?

---

# 11. Keys and Relationships

## Primary Key

Uniquely identifies a record.

```text
customer_id
order_id
payment_id
```

## Foreign Key

References another relational record.

```text
Order.customer_id -> Customer.customer_id
```

## Natural Key

A business value such as:

```text
email
product_sku
country_code
```

## Surrogate Key

A generated identifier such as:

```text
UUID
auto-increment ID
time-sortable distributed ID
```

## Relationship Types

```text
One-to-One
One-to-Many
Many-to-Many
```

For many-to-many relationships, relational systems often use a junction table.

```text
User
UserRole
Role
```

---

# 12. Design for Access Patterns

A schema must support important queries.

Examples:

```text
Get an order by order ID.

Get the latest 50 orders for a customer.

Find pending payments created today.

Find products by category and price range.
```

Access patterns determine:

- Table structure
- Document boundaries
- Partition keys
- Sort keys
- Secondary indexes
- Denormalized views

A schema that stores data correctly but cannot serve required queries efficiently is incomplete.

---

# 13. Indexing

Indexes improve reads but increase:

- Storage
- Write cost
- Maintenance
- Memory usage

Example query:

```sql
SELECT *
FROM orders
WHERE customer_id = ?
ORDER BY created_at DESC;
```

Useful index:

```text
(customer_id, created_at DESC)
```

For:

```text
WHERE customer_id = ?
AND status = ?
ORDER BY created_at DESC
```

A possible index is:

```text
(customer_id, status, created_at DESC)
```

Index order should match filtering and sorting patterns.

Avoid over-indexing because it can slow inserts and updates.

---

# 14. Normalization

Normalization separates data to reduce duplication.

Example:

```text
Customer
Order
OrderItem
Product
```

Advantages:

- Consistent updates
- Less duplication
- Strong integrity
- Smaller transactional records

Best for:

- Transactional systems
- Frequently changing shared data
- Complex relationships

---

# 15. Denormalization

Denormalization duplicates or precomputes data to improve reads.

Example:

```text
Order stores customer_name at order creation time.
```

Advantages:

- Fewer joins
- Faster reads
- Better distributed access
- Query-specific optimization

Limitations:

- Duplicate data
- Update complexity
- Eventual consistency
- More storage

Use denormalization intentionally for known access patterns.

---

# 16. Scaling and Sharding

Scale progressively:

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

## Vertical Scaling

Increase CPU, memory, or storage of one server.

## Horizontal Scaling

Add more nodes.

## Replication

Creates copies for:

- Availability
- Read scaling
- Disaster recovery

## Sharding

Distributes data across nodes.

Possible shard keys:

- User ID
- Tenant ID
- Region
- Time
- Hash of entity ID

A good shard key:

- Has high cardinality
- Distributes load evenly
- Matches access patterns
- Avoids hot partitions
- Limits cross-shard queries

---

# 17. Common Selection Scenarios

| Scenario | Likely Choice |
|---|---|
| Payment ledger | Relational |
| Product catalog with varying attributes | Document |
| Session storage | Key-value |
| Billions of telemetry records | Wide-column |
| Fraud relationship traversal | Graph |
| Order system | Relational, possibly with cache and search |
| Social feed | Wide-column or key-value plus relational source of truth |
| Recommendation graph | Graph plus other operational stores |

Many real systems use **polyglot persistence**: different databases for different workloads.

---

# 18. Senior Interview Questions

- How do you choose between SQL and NoSQL?
- Why should schema design begin with access patterns?
- When should data be embedded in a document?
- When should data be referenced?
- What makes a good partition key?
- What causes hot partitions?
- What is the difference between partitioning and sharding?
- What is normalization?
- When should you denormalize?
- How do composite indexes work?
- Why can too many indexes be harmful?
- How do read replicas affect consistency?
- What is replication lag?
- When would you choose a graph database?
- Why are joins avoided in Cassandra-like systems?
- What is query-first data modeling?
- What is polyglot persistence?
- How do you migrate a schema without downtime?
- How do you handle cross-shard transactions?
- How do you choose between UUID and auto-increment keys?

---

# 19. Interview Readiness Checklist

## Requirements

- [ ] I begin with requirements and access patterns.
- [ ] I clarify scale, consistency, and transactions.
- [ ] I identify read/write ratio.
- [ ] I identify retention and growth.

## Data Modeling

- [ ] I identify entities.
- [ ] I define keys.
- [ ] I explain relationships.
- [ ] I identify aggregate boundaries.
- [ ] I identify sensitive data.

## Database Models

- [ ] I can explain relational databases.
- [ ] I can explain document databases.
- [ ] I can explain key-value stores.
- [ ] I can explain wide-column databases.
- [ ] I can explain graph databases.
- [ ] I can compare their strengths and limitations.

## Access and Indexing

- [ ] I list critical queries.
- [ ] I design indexes for those queries.
- [ ] I understand composite index order.
- [ ] I understand index write cost.
- [ ] I avoid over-indexing.

## Normalization

- [ ] I can explain normalization.
- [ ] I can explain denormalization.
- [ ] I can justify duplication.
- [ ] I understand consistency implications.

## Scaling

- [ ] I understand replication.
- [ ] I understand partitioning.
- [ ] I understand sharding.
- [ ] I can choose a shard key.
- [ ] I can explain hot partitions.
- [ ] I can discuss cross-shard queries.
- [ ] I can explain replication lag.

---

# Senior Interview Summary

> I choose a database by starting with business requirements and access patterns rather than technology preference. Relational databases are my default when transactions, constraints, joins, and strong consistency are important. Document databases are useful when data is aggregate-oriented, nested, and evolves frequently. Key-value stores are ideal for direct lookups such as sessions, caches, counters, and idempotency keys. Wide-column databases fit very large distributed workloads with predictable partition-key queries and high write throughput. Graph databases are appropriate when multi-hop relationship traversal is the primary workload. For schema design, I identify entities, keys, relationships, and critical queries first. I then design indexes around those access patterns, normalize transactional data where consistency matters, and denormalize selectively for read performance. I scale progressively through query optimization, indexes, caching, replicas, partitioning, and finally sharding, while choosing partition keys that distribute traffic evenly and minimize cross-shard operations.
