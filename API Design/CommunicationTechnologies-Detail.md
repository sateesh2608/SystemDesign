# Application Communication Technologies — Detailed Interview Guide

This document provides a detailed understanding of:

- REST
- GraphQL
- gRPC
- Polling and Long Polling
- Server-Sent Events
- WebSockets
- WebRTC

The focus is on:

- How each technology works
- Important internal concepts
- Advantages and limitations
- Security
- Scalability
- Failure handling
- Practical use cases
- Common interview questions
- Interview readiness tracking

---

# Table of Contents

1. [REST](#1-rest)
2. [GraphQL](#2-graphql)
3. [gRPC](#3-grpc)
4. [Polling and Long Polling](#4-polling-and-long-polling)
5. [Server-Sent Events](#5-server-sent-events)
6. [WebSockets](#6-websockets)
7. [WebRTC](#7-webrtc)
8. [Technology Comparison](#8-technology-comparison)
9. [Scenario-Based Interview Questions](#9-scenario-based-interview-questions)
10. [Interview Readiness Tracker](#10-interview-readiness-tracker)

---

# 1. REST

## 1.1 What Is REST?

REST stands for **Representational State Transfer**.

REST is not a protocol or framework. It is an architectural style for designing network-based applications.

REST APIs typically use:

- HTTP as the communication protocol
- URLs to identify resources
- HTTP methods to perform operations
- JSON as the data format

Example:

```http
GET /customers/101
```

Response:

```json
{
  "id": 101,
  "name": "John",
  "email": "john@example.com"
}
```

The resource is:

```text
Customer
```

The representation is:

```text
JSON
```

---

## 1.2 Resource-Oriented Design

REST APIs should usually be designed around resources rather than actions.

Good:

```http
GET /customers/101
POST /customers
PUT /customers/101
DELETE /customers/101
```

Less RESTful:

```http
GET /getCustomer?id=101
POST /createCustomer
POST /deleteCustomer
```

Resource names should normally be nouns.

Recommended:

```text
/customers
/orders
/products
/payments
```

Avoid:

```text
/getCustomers
/createOrder
/deleteProduct
```

---

## 1.3 HTTP Methods

| Method | Purpose | Safe | Idempotent |
|---|---|---:|---:|
| GET | Retrieve a resource | Yes | Yes |
| HEAD | Retrieve headers only | Yes | Yes |
| OPTIONS | Discover supported operations | Yes | Yes |
| POST | Create or trigger processing | No | Usually no |
| PUT | Replace a resource | No | Yes |
| PATCH | Partially update a resource | No | Depends |
| DELETE | Delete a resource | No | Yes |

---

## 1.4 Safe Methods

A safe method should not modify the server state.

Examples:

```http
GET /customers/101
HEAD /customers/101
OPTIONS /customers
```

A GET request should not perform actions such as:

```http
GET /deleteCustomer/101
```

That operation changes state and should not use GET.

---

## 1.5 Idempotency

An operation is idempotent when executing it multiple times produces the same final result as executing it once.

Example:

```http
DELETE /customers/101
```

Calling this operation several times should leave the customer deleted.

Another example:

```http
PUT /customers/101
```

```json
{
  "name": "John"
}
```

Sending the same request repeatedly should leave the customer with the same name.

POST is generally not idempotent.

```http
POST /payments
```

Sending this request twice may create two payments.

For sensitive POST operations, an idempotency key can be used.

```http
POST /payments
Idempotency-Key: 8f14e45fceea
```

The server stores the result associated with the key and prevents duplicate processing.

---

## 1.6 PUT vs PATCH

### PUT

PUT normally replaces the complete resource.

```http
PUT /customers/101
```

```json
{
  "name": "John",
  "email": "john@example.com",
  "phone": "555-1000"
}
```

### PATCH

PATCH updates only selected fields.

```http
PATCH /customers/101
```

```json
{
  "phone": "555-2000"
}
```

PUT is generally idempotent.

PATCH can be idempotent depending on how it is implemented.

This PATCH may be idempotent:

```json
{
  "status": "ACTIVE"
}
```

This PATCH may not be idempotent:

```json
{
  "incrementBalanceBy": 100
}
```

---

## 1.7 HTTP Status Codes

### Successful responses

| Status | Meaning |
|---|---|
| 200 OK | Request completed successfully |
| 201 Created | Resource created successfully |
| 202 Accepted | Request accepted for asynchronous processing |
| 204 No Content | Request succeeded without a response body |

### Client errors

| Status | Meaning |
|---|---|
| 400 Bad Request | Invalid request |
| 401 Unauthorized | Authentication is missing or invalid |
| 403 Forbidden | User is authenticated but not authorized |
| 404 Not Found | Resource does not exist |
| 409 Conflict | Request conflicts with current state |
| 422 Unprocessable Content | Request is syntactically correct but semantically invalid |
| 429 Too Many Requests | Rate limit exceeded |

### Server errors

| Status | Meaning |
|---|---|
| 500 Internal Server Error | Unexpected server error |
| 502 Bad Gateway | Invalid response from an upstream service |
| 503 Service Unavailable | Service is temporarily unavailable |
| 504 Gateway Timeout | Upstream service did not respond in time |

---

## 1.8 401 vs 403

### 401 Unauthorized

The client is not successfully authenticated.

Examples:

- Missing token
- Expired token
- Invalid token

### 403 Forbidden

The client is authenticated but does not have permission.

Example:

```text
A normal user attempts to access an administrator endpoint.
```

---

## 1.9 Statelessness

REST requires requests to be stateless.

Each request must contain enough information for the server to process it.

Example:

```http
GET /orders/500
Authorization: Bearer <JWT>
```

The server should not depend on information from the previous request.

Benefits:

- Easier horizontal scaling
- Easier load balancing
- Better fault tolerance
- Any server instance can process the request

Stateless does not mean the application has no data.

It means the server does not rely on conversational client state between requests.

---

## 1.10 REST Constraints

The major REST constraints are:

### Client-server separation

The client and server evolve independently.

### Statelessness

Each request contains all required information.

### Cacheability

Responses specify whether they can be cached.

### Uniform interface

Resources are accessed through consistent interfaces.

### Layered system

The client may communicate through:

- Load balancers
- Gateways
- Proxies
- Security layers

### Code on demand

The server may optionally send executable code to the client.

This constraint is optional and is not commonly discussed in ordinary REST API implementations.

---

## 1.11 Caching

REST responses can use HTTP caching headers.

```http
Cache-Control: max-age=300
```

This tells the client or intermediary that the response can be cached for 300 seconds.

Other caching headers include:

```http
ETag: "customer-101-v5"
Last-Modified: Sat, 25 Jul 2026 10:00:00 GMT
```

The client can later send:

```http
If-None-Match: "customer-101-v5"
```

If the resource did not change, the server returns:

```http
304 Not Modified
```

This saves bandwidth because the response body does not need to be returned again.

---

## 1.12 Pagination

Large collections should not be returned in one response.

### Offset-based pagination

```http
GET /orders?page=2&size=20
```

Advantages:

- Simple
- Easy to implement
- Supports direct page access

Limitations:

- Performance can degrade with large offsets
- Data may shift when records are inserted or deleted

### Cursor-based pagination

```http
GET /orders?cursor=eyJpZCI6MTAwfQ&limit=20
```

Advantages:

- Better performance for large datasets
- More stable when records change
- Suitable for infinite scrolling

Limitations:

- More complex
- Direct page navigation is harder

---

## 1.13 Filtering and Sorting

Filtering:

```http
GET /orders?status=COMPLETED
```

Sorting:

```http
GET /orders?sort=createdAt,desc
```

Multiple filters:

```http
GET /orders?status=COMPLETED&customerId=101
```

The server should validate supported fields to prevent:

- Unexpected query behavior
- Performance issues
- Injection vulnerabilities

---

## 1.14 API Versioning

### URI versioning

```http
GET /api/v1/customers
GET /api/v2/customers
```

Advantages:

- Easy to understand
- Easy to route

### Header versioning

```http
Accept: application/vnd.company.v2+json
```

Advantages:

- Cleaner URLs

Limitations:

- Less visible
- Harder to test manually

### Query parameter versioning

```http
GET /customers?version=2
```

This is easy to implement but less commonly preferred for enterprise API design.

---

## 1.15 Error Response Design

A consistent error response helps clients handle failures.

```json
{
  "timestamp": "2026-07-25T15:30:00Z",
  "status": 400,
  "error": "Bad Request",
  "code": "INVALID_CUSTOMER_ID",
  "message": "Customer ID must be a positive number",
  "path": "/customers/-10",
  "correlationId": "ab12cd34"
}
```

Recommended fields:

- HTTP status
- Application-specific error code
- Human-readable message
- Request path
- Timestamp
- Correlation ID
- Validation details

Avoid exposing:

- Stack traces
- Database details
- Internal server paths
- Sensitive security information

---

## 1.16 REST Authentication and Authorization

Common approaches include:

- Basic Authentication
- Session-based authentication
- API keys
- OAuth 2.0
- OpenID Connect
- JWT access tokens
- Mutual TLS

Typical modern flow:

```text
User
  |
  v
Identity Provider
  |
  v
Access Token
  |
  v
REST API
```

Request:

```http
GET /orders
Authorization: Bearer eyJhbGciOi...
```

The API validates:

- Signature
- Issuer
- Audience
- Expiration
- Scopes
- Roles

---

## 1.17 REST Security Risks

Important risks include:

- Broken object-level authorization
- Injection attacks
- Missing authentication
- Excessive data exposure
- Mass assignment
- Improper rate limiting
- Insecure CORS configuration

Example of broken object-level authorization:

```http
GET /customers/101/orders
```

A user changes the customer ID:

```http
GET /customers/102/orders
```

The server must verify that the authenticated user is authorized to access customer 102.

It is not enough to validate only the token.

---

## 1.18 REST Performance Improvements

- Use caching
- Use pagination
- Compress responses
- Avoid returning unnecessary fields
- Use database indexes
- Use asynchronous processing for long-running operations
- Avoid excessive nested resources
- Configure timeouts
- Use connection pooling
- Use HTTP/2 when appropriate
- Apply rate limiting
- Use content delivery networks for cacheable public data

---

## 1.19 Asynchronous REST Processing

For long-running work, the API should avoid keeping the HTTP request open unnecessarily.

Example:

```http
POST /reports
```

Response:

```http
202 Accepted
Location: /reports/789/status
```

```json
{
  "jobId": 789,
  "status": "PROCESSING"
}
```

The client can check:

```http
GET /reports/789/status
```

When processing completes:

```json
{
  "jobId": 789,
  "status": "COMPLETED",
  "downloadUrl": "/reports/789/download"
}
```

---

## 1.20 REST Interview Questions

- What is REST?
- Is REST a protocol?
- What are REST constraints?
- What is statelessness?
- What is idempotency?
- Which HTTP methods are idempotent?
- What is the difference between PUT and PATCH?
- What is the difference between POST and PUT?
- What is the difference between 401 and 403?
- When would you return 202?
- How do you version REST APIs?
- How do you design pagination?
- Offset pagination vs cursor pagination?
- How do ETags work?
- How do you secure REST APIs?
- What is CORS?
- What is content negotiation?
- How do you handle duplicate payment requests?
- How do you design error responses?
- What is HATEOAS?
- How do you make REST APIs backward compatible?
- How do you handle long-running operations?
- What is rate limiting?
- What is throttling?
- How do you handle partial failures in REST-based microservices?

---

## 1.21 REST Interview Readiness

- [ ] I can explain REST in my own words.
- [ ] I can explain all major HTTP methods.
- [ ] I understand safety and idempotency.
- [ ] I can explain PUT vs PATCH.
- [ ] I know common HTTP status codes.
- [ ] I can explain 401 vs 403.
- [ ] I understand REST constraints.
- [ ] I can explain statelessness.
- [ ] I can design pagination.
- [ ] I can explain cursor-based pagination.
- [ ] I understand HTTP caching and ETags.
- [ ] I can explain API versioning approaches.
- [ ] I can design a consistent error response.
- [ ] I can explain OAuth 2.0 and JWT validation.
- [ ] I can explain idempotency keys.
- [ ] I can discuss REST security risks.
- [ ] I can explain asynchronous REST operations.
- [ ] I can design a production-ready REST API.

---

# 2. GraphQL

## 2.1 What Is GraphQL?

GraphQL is a query language and runtime for APIs.

It allows clients to specify exactly which fields they need.

A GraphQL API commonly exposes one endpoint:

```http
POST /graphql
```

The client sends a query describing the required data.

---

## 2.2 Why GraphQL?

Suppose a frontend page needs:

- Customer name
- Recent orders
- Payment status

With REST, it may call:

```http
GET /customers/101
GET /customers/101/orders
GET /customers/101/payments
```

With GraphQL:

```graphql
query {
  customer(id: 101) {
    name
    orders(limit: 5) {
      id
      amount
    }
    paymentStatus
  }
}
```

The client receives only the requested fields.

---

## 2.3 GraphQL Schema

GraphQL uses a strongly typed schema.

```graphql
type Customer {
  id: ID!
  name: String!
  email: String
  orders: [Order!]!
}

type Order {
  id: ID!
  amount: Float!
  status: OrderStatus!
}

enum OrderStatus {
  CREATED
  PROCESSING
  COMPLETED
  CANCELLED
}

type Query {
  customer(id: ID!): Customer
  orders(customerId: ID!): [Order!]!
}
```

Important symbols:

```text
String
```

The value may be null.

```text
String!
```

The value cannot be null.

```text
[Order]
```

The list and individual values may be null.

```text
[Order!]!
```

The list cannot be null, and its items cannot be null.

---

## 2.4 GraphQL Operation Types

### Query

Used to retrieve data.

```graphql
query {
  customer(id: 101) {
    id
    name
  }
}
```

### Mutation

Used to modify data.

```graphql
mutation {
  createCustomer(input: {
    name: "John"
    email: "john@example.com"
  }) {
    id
    name
  }
}
```

### Subscription

Used to receive real-time events.

```graphql
subscription {
  orderStatusChanged(orderId: 500) {
    orderId
    status
  }
}
```

Subscriptions are commonly transported over WebSockets.

---

## 2.5 Resolvers

A resolver is a function that provides data for a GraphQL field.

Example flow:

```text
GraphQL Query
      |
      v
Query Resolver
      |
      v
Customer Service
      |
      v
Database
```

For this query:

```graphql
query {
  customer(id: 101) {
    name
    orders {
      id
      amount
    }
  }
}
```

GraphQL may invoke:

- A customer resolver
- A name field resolver
- An orders resolver
- Resolvers for order fields

Simple fields may be resolved automatically from objects.

Complex fields usually require custom resolvers.

---

## 2.6 GraphQL Variables

Avoid building queries through string concatenation.

Query:

```graphql
query GetCustomer($customerId: ID!) {
  customer(id: $customerId) {
    id
    name
  }
}
```

Variables:

```json
{
  "customerId": "101"
}
```

Benefits:

- Safer
- Easier caching
- Easier reuse
- Cleaner client code

---

## 2.7 Fragments

Fragments allow reusable field selections.

```graphql
fragment CustomerSummary on Customer {
  id
  name
  email
}
```

Usage:

```graphql
query {
  customer(id: 101) {
    ...CustomerSummary
  }
}
```

---

## 2.8 N+1 Query Problem

Consider:

```graphql
query {
  customers {
    id
    orders {
      id
    }
  }
}
```

The server executes:

```text
1 query to load customers
+
1 order query per customer
```

For 100 customers:

```text
1 + 100 = 101 queries
```

This is the N+1 problem.

Solution:

- Batch requests
- Use DataLoader
- Fetch joins carefully
- Use prefetching
- Cache repeated resolver requests

DataLoader can collect customer IDs and execute one batch query:

```sql
SELECT *
FROM orders
WHERE customer_id IN (1, 2, 3, 4, 5);
```

---

## 2.9 Over-Fetching Is Reduced, Not Automatically Eliminated

GraphQL allows the client to request specific fields.

However, the backend may still over-fetch from the database.

Example:

The GraphQL client requests:

```graphql
customer {
  name
}
```

But the repository executes:

```sql
SELECT *
FROM customer;
```

GraphQL controls the API response shape, but backend implementation still determines database efficiency.

---

## 2.10 GraphQL Error Handling

GraphQL commonly returns HTTP 200 even when some fields fail.

Example:

```json
{
  "data": {
    "customer": {
      "name": "John",
      "orders": null
    }
  },
  "errors": [
    {
      "message": "Order service unavailable",
      "path": ["customer", "orders"]
    }
  ]
}
```

GraphQL supports partial responses.

This is useful when one field fails but other fields can still be returned.

Applications should define:

- Standard error codes
- User-friendly messages
- Correlation IDs
- Retryability information
- Field-specific errors

---

## 2.11 GraphQL Authentication and Authorization

Authentication normally happens before query execution.

Example:

```http
Authorization: Bearer <JWT>
```

Authorization must still be applied at:

- Query level
- Mutation level
- Field level
- Object level

Example:

A user may access a customer object but should not access:

```graphql
customer {
  salary
}
```

Field-level authorization may be necessary.

Resolvers should not assume that because the parent object was authorized, every field is authorized.

---

## 2.12 Query Complexity and Depth

A malicious or poorly designed query may request deeply nested data.

```graphql
query {
  customers {
    orders {
      customer {
        orders {
          customer {
            orders {
              id
            }
          }
        }
      }
    }
  }
}
```

Protection techniques:

- Maximum query depth
- Query complexity scoring
- Request timeout
- Maximum result size
- Pagination requirements
- Rate limiting
- Persisted queries
- Resolver-level limits

---

## 2.13 GraphQL Caching

REST can cache resources by URL.

GraphQL often uses one endpoint:

```http
POST /graphql
```

This makes traditional HTTP caching less straightforward.

Caching approaches include:

- Client-side normalized caching
- Resolver caching
- DataLoader request caching
- Persisted query caching
- CDN caching for known operations
- Application-level caching

Clients such as Apollo may normalize data by object type and ID.

Example cache key:

```text
Customer:101
```

---

## 2.14 Persisted Queries

Instead of sending the entire query every time, the client sends a query identifier or hash.

```json
{
  "queryHash": "abc123",
  "variables": {
    "customerId": 101
  }
}
```

Benefits:

- Smaller requests
- Better security control
- Easier allow-listing
- Improved CDN compatibility
- Prevents arbitrary queries in controlled environments

---

## 2.15 Schema Evolution

GraphQL normally avoids endpoint versions.

Instead, fields are added gradually.

Old field:

```graphql
type Customer {
  fullName: String
}
```

New preferred fields:

```graphql
type Customer {
  firstName: String
  lastName: String
  fullName: String @deprecated(reason: "Use firstName and lastName")
}
```

Clients can migrate over time.

Once no clients use the deprecated field, it may be removed.

---

## 2.16 GraphQL Federation

In large systems, different teams may own parts of the graph.

Example:

```text
Customer Subgraph
Order Subgraph
Payment Subgraph
Product Subgraph
```

A federation gateway combines them into one graph.

```text
Client
   |
   v
GraphQL Gateway
   |
   +--> Customer Subgraph
   +--> Order Subgraph
   +--> Payment Subgraph
```

Benefits:

- Team ownership
- Independent development
- Unified API to clients

Challenges:

- Distributed tracing
- Gateway dependency
- Schema coordination
- Cross-service performance
- Authorization consistency

---

## 2.17 GraphQL Advantages

- Clients request only required fields
- Strongly typed schema
- Good developer experience
- Supports multiple related resources in one operation
- Supports schema introspection
- Supports gradual schema evolution
- Good fit for complex user interfaces
- Supports partial responses

---

## 2.18 GraphQL Limitations

- Server implementation can be complex
- N+1 query problems
- Query cost can be unpredictable
- Traditional HTTP caching is less direct
- File uploads need additional conventions
- Monitoring requires operation-level visibility
- Authorization can become complicated
- Schema design requires governance
- May be unnecessary for simple CRUD systems

---

## 2.19 When to Use GraphQL

Use GraphQL when:

- Different clients need different response shapes
- Frontend screens aggregate many related resources
- Mobile clients need to minimize payloads
- Product requirements change frequently
- A unified API layer is valuable
- Multiple teams contribute to a shared domain graph

Avoid adding GraphQL only because it is popular.

REST may be simpler when:

- APIs are small
- Response structures are stable
- Standard HTTP caching is important
- External consumers expect conventional REST APIs
- The team lacks GraphQL operational experience

---

## 2.20 GraphQL Interview Questions

- What is GraphQL?
- How is GraphQL different from REST?
- What are queries, mutations and subscriptions?
- What is a GraphQL schema?
- What is a resolver?
- What is the N+1 problem?
- How does DataLoader solve N+1?
- How do you secure GraphQL APIs?
- How do you implement field-level authorization?
- How do you prevent expensive queries?
- What is query depth?
- What is query complexity?
- Why can GraphQL return HTTP 200 with errors?
- What is a partial response?
- How do you cache GraphQL responses?
- What are persisted queries?
- How do you version GraphQL APIs?
- What is schema deprecation?
- What is GraphQL federation?
- When would you choose REST over GraphQL?
- How are subscriptions implemented?
- What are the risks of schema introspection in production?
- How do you monitor resolver performance?

---

## 2.21 GraphQL Interview Readiness

- [ ] I can explain GraphQL in my own words.
- [ ] I can compare REST and GraphQL.
- [ ] I understand schemas and types.
- [ ] I can explain nullability.
- [ ] I understand queries, mutations and subscriptions.
- [ ] I can explain resolvers.
- [ ] I understand the N+1 problem.
- [ ] I can explain DataLoader and batching.
- [ ] I understand partial responses.
- [ ] I can explain GraphQL error handling.
- [ ] I can discuss authentication and authorization.
- [ ] I can explain query depth and complexity.
- [ ] I can describe caching approaches.
- [ ] I understand persisted queries.
- [ ] I can explain schema evolution and deprecation.
- [ ] I understand GraphQL federation.
- [ ] I can explain when GraphQL is unnecessary.
- [ ] I can design a production-ready GraphQL API.

---

# 3. gRPC

## 3.1 What Is gRPC?

gRPC is a high-performance Remote Procedure Call framework.

It commonly uses:

- HTTP/2
- Protocol Buffers
- Generated client and server code
- Strongly typed service contracts

Instead of thinking primarily in terms of URLs and resources, gRPC models remote methods.

Example:

```text
InventoryService.checkStock()
```

---

## 3.2 Protocol Buffers

Protocol Buffers define messages and services in `.proto` files.

```proto
syntax = "proto3";

package inventory;

service InventoryService {
  rpc CheckStock(ProductRequest) returns (ProductResponse);
}

message ProductRequest {
  string product_id = 1;
}

message ProductResponse {
  string product_id = 1;
  int32 available_quantity = 2;
  bool available = 3;
}
```

The numbers:

```proto
product_id = 1;
available_quantity = 2;
```

are field identifiers used in binary serialization.

They should not be changed or reused carelessly after a schema is released.

---

## 3.3 Code Generation

The protobuf compiler generates language-specific code.

From one contract, it can generate:

- Java classes
- Go structs
- Python classes
- C# classes
- Client stubs
- Server interfaces

This reduces manual serialization and contract mismatch.

---

## 3.4 gRPC Call Types

### Unary RPC

One request and one response.

```text
Client
  |
  | Request
  v
Server
  |
  | Response
  v
Client
```

Example:

```proto
rpc CheckStock(ProductRequest) returns (ProductResponse);
```

### Server streaming

One request and multiple responses.

```text
Client
  |
  | Request
  v
Server
  |
  | Response 1
  | Response 2
  | Response 3
  v
Client
```

Example:

```proto
rpc TrackOrder(OrderRequest) returns (stream OrderUpdate);
```

### Client streaming

Multiple requests and one response.

```text
Client
  |
  | Message 1
  | Message 2
  | Message 3
  v
Server
  |
  | Final Response
  v
Client
```

Example:

```proto
rpc UploadMetrics(stream Metric) returns (UploadSummary);
```

### Bidirectional streaming

Both client and server send streams independently.

```text
Client
   ⇅
Server
```

Example:

```proto
rpc Chat(stream ChatMessage) returns (stream ChatMessage);
```

---

## 3.5 Why HTTP/2 Matters

HTTP/2 provides:

- Multiplexing
- Header compression
- Long-lived connections
- Binary framing
- Stream prioritization

With multiplexing, multiple RPC calls can share one TCP connection.

```text
Single HTTP/2 Connection
    |
    +-- RPC Stream 1
    +-- RPC Stream 2
    +-- RPC Stream 3
```

This reduces the need for many separate connections.

---

## 3.6 Deadlines

A deadline tells the server how long the client is willing to wait.

Example:

```text
Client deadline: 500 milliseconds
```

If processing exceeds the deadline, the call is cancelled.

Deadlines prevent requests from waiting indefinitely and consuming resources.

Every production gRPC call should normally have a deadline.

---

## 3.7 Timeouts vs Deadlines

A timeout describes a duration.

```text
Wait for 2 seconds
```

A deadline identifies the maximum completion time.

```text
Complete before 10:30:02
```

gRPC internally propagates deadline information so downstream services can avoid continuing work after the caller no longer needs the response.

---

## 3.8 Cancellation

If a client cancels the request, the server should detect the cancellation and stop unnecessary work.

Example:

```text
User leaves the page
      |
      v
Client cancels gRPC request
      |
      v
Server stops database or downstream processing
```

Cancellation must be respected by application code where possible.

---

## 3.9 Metadata

Metadata is similar to HTTP headers.

It can carry:

- Authentication tokens
- Correlation IDs
- Tenant IDs
- Tracing information
- Locale
- Client version

Example:

```text
authorization: Bearer <token>
x-correlation-id: abc123
```

Interceptors can process metadata consistently across RPC calls.

---

## 3.10 gRPC Interceptors

Interceptors are similar to filters or middleware.

They can handle:

- Authentication
- Logging
- Metrics
- Tracing
- Validation
- Exception conversion
- Retry logic

```text
Client
  |
Client Interceptor
  |
Network
  |
Server Interceptor
  |
Service Method
```

---

## 3.11 gRPC Status Codes

Common gRPC status codes include:

| Code | Meaning |
|---|---|
| OK | Successful operation |
| CANCELLED | Request was cancelled |
| INVALID_ARGUMENT | Invalid input |
| DEADLINE_EXCEEDED | Deadline expired |
| NOT_FOUND | Resource was not found |
| ALREADY_EXISTS | Resource already exists |
| PERMISSION_DENIED | Caller lacks permission |
| UNAUTHENTICATED | Authentication failed |
| RESOURCE_EXHAUSTED | Rate limit or capacity exceeded |
| FAILED_PRECONDITION | System state does not allow the operation |
| ABORTED | Operation was aborted due to conflict |
| INTERNAL | Internal server failure |
| UNAVAILABLE | Service is temporarily unavailable |

Applications should map domain errors to appropriate gRPC status codes.

---

## 3.12 Schema Compatibility

Protocol Buffer schemas should evolve carefully.

Safe changes usually include:

- Adding new fields
- Adding new message types
- Adding new RPC methods
- Adding enum values carefully

Avoid:

- Reusing deleted field numbers
- Changing field types incompatibly
- Renaming fields while changing their semantic meaning
- Removing fields still used by clients

Reserve removed fields:

```proto
message Customer {
  reserved 3;
  reserved "old_email";
}
```

This prevents accidental reuse.

---

## 3.13 gRPC Authentication

Common approaches:

- TLS
- Mutual TLS
- OAuth 2.0 access tokens
- JWT metadata
- Service identities
- Cloud workload identities

Example:

```text
Service A
   |
   | Mutual TLS + token
   v
Service B
```

TLS protects communication.

The token or certificate identity determines who is calling.

---

## 3.14 Load Balancing

gRPC uses long-lived HTTP/2 connections.

Traditional server-side load balancing may route the entire connection to one backend.

This can cause uneven traffic distribution.

Approaches include:

- Client-side load balancing
- Service mesh
- DNS-based discovery
- Proxy-aware gRPC load balancing
- Envoy
- Cloud-native gRPC-aware load balancers

The load balancer must support HTTP/2 and gRPC correctly.

---

## 3.15 Retries

Retries are useful only for safe and transient failures.

Potential retryable errors:

- UNAVAILABLE
- RESOURCE_EXHAUSTED in selected scenarios
- Some deadline failures when the remaining operation is safe

Avoid retrying:

- Validation failures
- Authentication failures
- Non-idempotent operations without protection

Use:

- Limited retry attempts
- Exponential backoff
- Jitter
- Deadlines
- Idempotency controls

---

## 3.16 gRPC-Web

Browsers do not directly support all native gRPC capabilities.

gRPC-Web allows browser applications to communicate with gRPC backends through a compatible proxy or server.

```text
Browser
   |
gRPC-Web
   |
Proxy
   |
Native gRPC Service
```

Limitations may apply compared with native gRPC, especially for some streaming modes.

---

## 3.17 gRPC vs REST

| Area | REST | gRPC |
|---|---|---|
| Data format | Usually JSON | Protocol Buffers |
| Protocol | HTTP/1.1 or HTTP/2 | HTTP/2 |
| Human readability | High | Low |
| Browser support | Excellent | Limited without gRPC-Web |
| Contract | OpenAPI optional | `.proto` required |
| Performance | Good | Usually better for internal high-volume calls |
| Streaming | Limited or separate mechanisms | Built-in |
| Public API suitability | Strong | Less common |
| Internal microservices | Good | Strong |

---

## 3.18 When to Use gRPC

Use gRPC when:

- Services communicate internally
- Low latency is important
- Payload efficiency matters
- Strong contracts are required
- Streaming is required
- Services use multiple programming languages
- The organization supports protobuf governance

REST may be better when:

- The API is public
- Browser compatibility is essential
- Human-readable payloads are important
- Consumers need simple curl-based debugging
- Infrastructure does not support HTTP/2 correctly

---

## 3.19 gRPC Interview Questions

- What is gRPC?
- How does gRPC differ from REST?
- What are Protocol Buffers?
- Why are protobuf field numbers important?
- What are the four gRPC communication types?
- What are deadlines?
- How are deadlines different from timeouts?
- How does cancellation work?
- What is gRPC metadata?
- What are interceptors?
- How do you secure gRPC communication?
- What is mutual TLS?
- How does gRPC load balancing work?
- Why can long-lived connections affect load distribution?
- How do retries work in gRPC?
- Which operations should not be retried?
- What is gRPC-Web?
- How do you maintain protobuf backward compatibility?
- What are common gRPC status codes?
- What is bidirectional streaming?
- How do you implement observability for gRPC?
- When would you choose REST over gRPC?

---

## 3.20 gRPC Interview Readiness

- [ ] I can explain gRPC in my own words.
- [ ] I understand Protocol Buffers.
- [ ] I can write a basic `.proto` contract.
- [ ] I understand protobuf field numbers.
- [ ] I can explain all four RPC types.
- [ ] I understand HTTP/2 multiplexing.
- [ ] I can explain deadlines and cancellation.
- [ ] I understand metadata and interceptors.
- [ ] I know common gRPC status codes.
- [ ] I can explain authentication and mutual TLS.
- [ ] I understand load balancing challenges.
- [ ] I can explain safe retry behavior.
- [ ] I understand schema compatibility.
- [ ] I know what gRPC-Web is.
- [ ] I can compare REST and gRPC.
- [ ] I can explain when gRPC is not a good choice.
- [ ] I can design a production-ready gRPC service.

---

# 4. Polling and Long Polling

## 4.1 What Is Polling?

Polling is a communication pattern in which the client repeatedly asks the server whether new data is available.

```text
Client
  |
  | GET /status
  v
Server
  |
  | No change
  v
Client

Wait 5 seconds and repeat
```

Example:

```javascript
setInterval(async () => {
  const response = await fetch("/api/orders/500/status");
  const data = await response.json();
  console.log(data.status);
}, 5000);
```

---

## 4.2 Polling Interval

The polling interval determines how frequently the client sends requests.

Example:

```text
Every 2 seconds
Every 10 seconds
Every 1 minute
```

A short interval provides faster updates but creates more load.

A long interval reduces load but increases update delay.

---

## 4.3 Polling Load Calculation

Suppose:

```text
10,000 clients
```

Each client polls every:

```text
5 seconds
```

Requests per minute:

```text
10,000 × 12 = 120,000 requests per minute
```

Most responses may contain no changes.

This creates unnecessary:

- Network traffic
- CPU usage
- Database queries
- Logging
- Authentication validation
- Load-balancer processing

---

## 4.4 Conditional Polling

Polling can be improved using ETags.

Initial response:

```http
ETag: "order-500-v10"
```

Next request:

```http
If-None-Match: "order-500-v10"
```

If the status did not change:

```http
304 Not Modified
```

The server still handles the request, but it avoids sending the full response body.

---

## 4.5 Adaptive Polling

Instead of polling at a fixed rate, the client can adjust the interval.

Example:

```text
First 30 seconds: poll every 2 seconds
Next 2 minutes: poll every 10 seconds
Later: poll every 30 seconds
```

This is useful when updates are more likely immediately after an operation starts.

---

## 4.6 Exponential Backoff

When failures occur, the client should not retry continuously.

Example:

```text
Attempt 1: wait 1 second
Attempt 2: wait 2 seconds
Attempt 3: wait 4 seconds
Attempt 4: wait 8 seconds
```

A maximum wait duration should normally be configured.

---

## 4.7 Jitter

If thousands of clients retry at exactly the same time, the server may experience a retry storm.

Jitter adds randomness.

Instead of all clients retrying after exactly 8 seconds:

```text
Client A: 7.3 seconds
Client B: 8.8 seconds
Client C: 6.9 seconds
```

This spreads the load.

---

## 4.8 Long Polling

Long polling is an improvement over regular polling.

The client sends a request.

The server keeps the request open until:

- New data becomes available
- A timeout occurs

```text
Client
  |
  | GET /notifications
  v
Server
  |
  | Wait until event occurs
  |
  | Return event
  v
Client
  |
  | Immediately send another request
  v
Server
```

---

## 4.9 Long Polling Flow

1. Client sends a request.
2. Server checks for available data.
3. If data exists, the server responds immediately.
4. If no data exists, the request remains open.
5. When data arrives, the server responds.
6. Client processes the response.
7. Client creates a new request.

---

## 4.10 Polling vs Long Polling

| Area | Polling | Long Polling |
|---|---|---|
| Request behavior | Sent at fixed intervals | Held until data or timeout |
| Empty responses | Common | Reduced |
| Update latency | Depends on interval | Usually lower |
| Server complexity | Low | Medium |
| Open requests | Short-lived | Long-lived |
| Scalability | Can create many requests | Requires connection management |

---

## 4.11 Polling Advantages

- Easy to implement
- Works with normal HTTP
- Compatible with browsers and proxies
- Easy to secure
- No permanent connection
- Suitable for infrequent updates

---

## 4.12 Polling Limitations

- Unnecessary requests
- Delayed updates
- Increased server load
- Increased database load
- Retry synchronization risk
- Poor fit for high-frequency real-time updates
- Mobile battery and data usage can increase

---

## 4.13 When Polling Is Appropriate

Polling is reasonable when:

- Updates are infrequent
- A delay of several seconds or minutes is acceptable
- Implementation simplicity matters
- The number of clients is limited
- Infrastructure cannot support persistent connections
- A background job status must be checked
- The page is not permanently open

Examples:

- Report generation status
- Batch processing status
- Payment reconciliation status
- Order status
- Legacy monitoring screen

---

## 4.14 Polling Interview Questions

- What is polling?
- What is the difference between polling and long polling?
- Why is frequent polling inefficient?
- How do you choose a polling interval?
- What is adaptive polling?
- What is exponential backoff?
- Why is jitter important?
- How can ETags improve polling?
- When would polling be better than WebSockets?
- How do you prevent retry storms?
- How do you calculate polling load?
- What happens when the browser tab is in the background?
- How do you stop polling when the user leaves the page?
- How do you handle overlapping polling requests?
- What is the difference between long polling and SSE?

---

## 4.15 Polling Interview Readiness

- [ ] I can explain normal polling.
- [ ] I can calculate polling request volume.
- [ ] I understand polling intervals.
- [ ] I can explain adaptive polling.
- [ ] I understand exponential backoff.
- [ ] I understand jitter.
- [ ] I can explain conditional requests and ETags.
- [ ] I understand long polling.
- [ ] I can compare polling and long polling.
- [ ] I can explain when polling is acceptable.
- [ ] I can discuss polling scalability.
- [ ] I can explain how to prevent overlapping requests.
- [ ] I can compare polling with SSE and WebSockets.

---

# 5. Server-Sent Events

## 5.1 What Is SSE?

Server-Sent Events allow a server to push events to a client over a long-lived HTTP connection.

Communication is one-way:

```text
Server
  |
  v
Client
```

The browser opens a connection using the `EventSource` API.

```javascript
const source = new EventSource("/api/notifications");

source.onmessage = event => {
  console.log(event.data);
};
```

---

## 5.2 SSE Response Type

The server returns:

```http
Content-Type: text/event-stream
Cache-Control: no-cache
Connection: keep-alive
```

The response remains open.

Events are sent as text.

```text
data: Order completed

```

The blank line marks the end of an event.

---

## 5.3 SSE Event Format

An SSE event may contain:

```text
id: 1001
event: order-status
retry: 5000
data: {"orderId":500,"status":"COMPLETED"}

```

Fields:

### id

Identifies the event.

```text
id: 1001
```

### event

Specifies the event type.

```text
event: order-status
```

### data

Contains the event payload.

```text
data: {"orderId":500,"status":"COMPLETED"}
```

### retry

Suggests a reconnection delay in milliseconds.

```text
retry: 5000
```

---

## 5.4 Named Events

Server event:

```text
event: payment-completed
data: {"paymentId":123}

```

Client:

```javascript
source.addEventListener("payment-completed", event => {
  const payment = JSON.parse(event.data);
  console.log(payment.paymentId);
});
```

---

## 5.5 Automatic Reconnection

If the connection is interrupted, the browser automatically attempts to reconnect.

This is one of SSE's main advantages over manually implemented streaming.

The server can send:

```text
retry: 3000
```

This requests a reconnection delay of approximately three seconds.

---

## 5.6 Last-Event-ID

When reconnecting, the client may send the last event ID it received.

```http
Last-Event-ID: 1001
```

The server can resume from the next event.

```text
1002
1003
1004
```

This helps prevent missed updates.

The server must maintain enough event history to support replay.

---

## 5.7 Heartbeats

Proxies, gateways or load balancers may close idle connections.

The server can send periodic heartbeat comments.

```text
: heartbeat

```

Lines beginning with `:` are comments.

They keep the connection active without creating a normal application event.

---

## 5.8 SSE Architecture

```text
Application Event
      |
      v
Message Broker
      |
      v
SSE Service
      |
      v
Connected Browser
```

For multiple server instances:

```text
Service Instance A
Service Instance B
Service Instance C
        |
        v
Shared Pub/Sub System
        |
        v
SSE Connections
```

A shared event system may use:

- Redis Pub/Sub
- Kafka
- Cloud messaging services
- Internal event bus

---

## 5.9 SSE Authentication

The native browser `EventSource` API has limitations for adding custom headers.

Possible approaches:

- Secure cookies
- Same-origin authentication
- Token in a short-lived query parameter
- Fetch-based streaming implementations
- Reverse proxy authentication
- Create a temporary signed connection URL

Avoid placing long-lived sensitive tokens in URLs because URLs may appear in:

- Logs
- Browser history
- Monitoring systems
- Proxy logs

---

## 5.10 SSE Backpressure

A slow client may consume events more slowly than the server produces them.

The server must decide whether to:

- Buffer events
- Drop old events
- Disconnect the client
- Send only the latest state
- Store events for replay

The correct strategy depends on whether every event matters.

Examples:

For CPU monitoring:

```text
Only the latest value may matter.
```

For financial transactions:

```text
Every event may matter.
```

SSE itself does not automatically solve application-level backpressure.

---

## 5.11 SSE Scaling Challenges

Each connected client keeps an HTTP connection open.

The server must manage:

- File descriptors
- Memory
- Connection timeouts
- Load-balancer limits
- Proxy buffering
- Reconnection storms
- Per-user authorization
- Event fan-out

For horizontal scaling, avoid relying only on in-memory events from one instance.

---

## 5.12 SSE vs Polling

| Area | Polling | SSE |
|---|---|---|
| Updates | Client repeatedly asks | Server pushes |
| Latency | Depends on interval | Usually low |
| Requests | Repeated | One long-lived connection |
| Empty responses | Common | Avoided |
| Reconnect | Client logic | Built into EventSource |
| Complexity | Lower | Moderate |

---

## 5.13 SSE vs WebSockets

| Area | SSE | WebSockets |
|---|---|---|
| Direction | Server to client | Bidirectional |
| Protocol | HTTP streaming | WebSocket protocol |
| Browser API | EventSource | WebSocket |
| Message format | Text | Text or binary |
| Automatic reconnect | Built in | Must be implemented |
| Best use | Notifications and feeds | Interactive real-time communication |
| Complexity | Lower | Higher |

---

## 5.14 When to Use SSE

Use SSE when:

- Only the server pushes updates
- Text-based events are sufficient
- Automatic reconnect is valuable
- HTTP infrastructure is preferred
- Updates are frequent enough to make polling inefficient

Examples:

- Notifications
- Live dashboards
- Deployment logs
- Order status
- News feeds
- Job progress
- Monitoring metrics
- AI response streaming

---

## 5.15 SSE Limitations

- Primarily server-to-client
- Native EventSource uses text
- Custom header support is limited
- Persistent connections require infrastructure support
- Proxy buffering may delay events
- Browser connection limits may matter in some architectures
- Application-level event replay must be designed
- Not ideal for high-frequency bidirectional messaging

---

## 5.16 SSE Interview Questions

- What are Server-Sent Events?
- How does SSE work?
- What is `text/event-stream`?
- What fields can an SSE event contain?
- How does SSE reconnect automatically?
- What is `Last-Event-ID`?
- How do you avoid losing events?
- Why are heartbeats needed?
- How do you authenticate an SSE connection?
- How do you scale SSE across multiple servers?
- What happens when a client is slow?
- How do you handle backpressure?
- SSE vs polling?
- SSE vs long polling?
- SSE vs WebSockets?
- Can SSE send binary data?
- What happens when a load balancer closes idle connections?
- When would SSE be a better choice than WebSockets?

---

## 5.17 SSE Interview Readiness

- [ ] I can explain SSE in my own words.
- [ ] I understand `text/event-stream`.
- [ ] I can explain the SSE event format.
- [ ] I know how named events work.
- [ ] I understand automatic reconnection.
- [ ] I can explain `Last-Event-ID`.
- [ ] I understand heartbeats.
- [ ] I can discuss authentication options.
- [ ] I can explain horizontal scaling.
- [ ] I understand slow-client handling.
- [ ] I can compare SSE with polling.
- [ ] I can compare SSE with WebSockets.
- [ ] I can identify appropriate SSE use cases.
- [ ] I can design a production-ready SSE service.

---

# 6. WebSockets

## 6.1 What Are WebSockets?

WebSockets provide persistent, bidirectional communication between a client and server.

```text
Client
   ⇅
Server
```

After the connection is established, both sides can send messages at any time.

---

## 6.2 WebSocket Handshake

A WebSocket connection begins as an HTTP request.

Client request:

```http
GET /chat HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
```

Server response:

```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: <calculated-value>
```

After the upgrade, the connection uses WebSocket framing rather than normal HTTP request-response behavior.

---

## 6.3 WebSocket URLs

Unencrypted:

```text
ws://example.com/chat
```

Encrypted:

```text
wss://example.com/chat
```

Production systems should normally use:

```text
wss://
```

---

## 6.4 Browser Example

```javascript
const socket = new WebSocket("wss://example.com/chat");

socket.onopen = () => {
  socket.send(JSON.stringify({
    type: "JOIN_ROOM",
    roomId: "engineering"
  }));
};

socket.onmessage = event => {
  const message = JSON.parse(event.data);
  console.log(message);
};

socket.onerror = error => {
  console.error(error);
};

socket.onclose = event => {
  console.log("Connection closed", event.code);
};
```

---

## 6.5 WebSocket Frames

WebSocket communication is divided into frames.

Frame types include:

- Text
- Binary
- Ping
- Pong
- Close
- Continuation

Text frames contain UTF-8 data.

Binary frames can carry:

- Images
- Encoded messages
- Protobuf payloads
- Compressed application data

---

## 6.6 Ping and Pong

Ping and pong frames help:

- Verify that the connection is alive
- Detect broken connections
- Keep connections active through infrastructure
- Measure latency

```text
Server -- Ping --> Client
Server <-- Pong -- Client
```

Application-level heartbeats may also be implemented.

---

## 6.7 WebSocket Connection State

A connection commonly passes through:

```text
CONNECTING
    |
    v
OPEN
    |
    v
CLOSING
    |
    v
CLOSED
```

Applications must handle reconnection because browsers do not automatically reconnect WebSockets.

---

## 6.8 Reconnection Strategy

A good reconnect strategy includes:

- Exponential backoff
- Jitter
- Maximum retry interval
- Authentication refresh
- Resubscription
- Missed-message recovery
- Connection status visibility

Example:

```text
Attempt 1: 1 second
Attempt 2: 2 seconds
Attempt 3: 4 seconds
Attempt 4: 8 seconds plus jitter
```

After reconnecting, the client may need to send:

```json
{
  "type": "RESUME",
  "lastReceivedSequence": 10500
}
```

---

## 6.9 Message Ordering

TCP preserves byte ordering within one connection.

However, application-level ordering issues can still happen because:

- Messages may be processed asynchronously
- Multiple servers may publish events
- Reconnection may replay messages
- Background workers may complete out of order

Sequence numbers can help.

```json
{
  "sequence": 5001,
  "type": "ORDER_UPDATED",
  "payload": {}
}
```

The client can detect:

- Missing messages
- Duplicate messages
- Out-of-order messages

---

## 6.10 Delivery Guarantees

WebSockets provide transport, not a complete messaging guarantee.

Applications must decide whether delivery is:

- At most once
- At least once
- Effectively once through deduplication

For reliable delivery, use:

- Message IDs
- Acknowledgements
- Retry buffers
- Persistent event storage
- Deduplication
- Sequence tracking

Example acknowledgement:

```json
{
  "type": "ACK",
  "messageId": "msg-1001"
}
```

---

## 6.11 WebSocket Authentication

Possible approaches:

- Secure cookie during handshake
- Token in initial connection URL
- Subprotocol-based token exchange
- Authenticate with the first application message
- Short-lived signed connection URL

Example application authentication message:

```json
{
  "type": "AUTHENTICATE",
  "accessToken": "<short-lived-token>"
}
```

The server must also handle token expiration during long-lived connections.

Possible strategies:

- Disconnect when the token expires
- Allow token refresh over the connection
- Use short-lived session credentials
- Reauthenticate periodically

---

## 6.12 Authorization

Authentication answers:

```text
Who is connected?
```

Authorization answers:

```text
Which rooms, channels or resources can they access?
```

Example:

```json
{
  "type": "SUBSCRIBE",
  "channel": "customer-101-orders"
}
```

The server must verify that the connected user has access to customer 101.

Do not trust channel names supplied by the client.

---

## 6.13 Horizontal Scaling

Suppose a user is connected to server A.

Another user's update is processed by server B.

Server B must communicate the event to server A.

```text
User 1 --> Server A
User 2 --> Server B

Server B
   |
Shared Pub/Sub
   |
Server A
   |
User 1
```

Shared systems may include:

- Redis Pub/Sub
- Kafka
- Cloud messaging systems
- Dedicated real-time platforms

---

## 6.14 Sticky Sessions

Sticky sessions route the same client to the same server.

They can simplify connection management but do not solve cross-server event delivery.

Even with sticky sessions, a shared messaging layer may still be needed.

Limitations:

- Uneven load
- Harder failover
- Rebalancing challenges
- Server failure disconnects assigned clients

---

## 6.15 Connection Registry

Servers may maintain a registry such as:

```text
userId -> connectionId
roomId -> list of connectionIds
serverId -> active connections
```

For distributed systems, part of this registry may need to be shared or discoverable.

Avoid excessive writes to a central store for every minor connection event unless necessary.

---

## 6.16 Backpressure

A client may receive data more slowly than the server sends it.

Possible strategies:

- Buffer messages temporarily
- Drop noncritical updates
- Keep only the most recent state
- Apply per-client rate limits
- Disconnect slow consumers
- Use acknowledgements
- Pause upstream producers

Example:

For live mouse positions:

```text
Drop older positions and send the newest.
```

For payment events:

```text
Do not drop messages.
```

---

## 6.17 WebSocket Message Protocol

WebSocket defines transport, but the application still needs a message contract.

Example:

```json
{
  "type": "ORDER_STATUS_CHANGED",
  "version": 1,
  "messageId": "abc123",
  "timestamp": "2026-07-25T15:30:00Z",
  "payload": {
    "orderId": 500,
    "status": "COMPLETED"
  }
}
```

Important fields:

- Message type
- Version
- Message ID
- Timestamp
- Payload
- Correlation ID

---

## 6.18 WebSocket Subprotocols

Applications may use subprotocols such as:

- STOMP
- GraphQL over WebSocket
- Custom JSON protocol
- Custom binary protocol

A subprotocol defines application-level semantics on top of WebSocket transport.

---

## 6.19 WebSocket Security Risks

Important risks include:

- Missing origin validation
- Cross-site WebSocket hijacking
- Weak authentication
- Unauthorized channel subscriptions
- Message flooding
- Large payload attacks
- Connection exhaustion
- Sensitive data leakage
- Missing rate limits

Controls include:

- Validate the `Origin` header
- Use `wss://`
- Authenticate every connection
- Authorize every subscription
- Limit payload size
- Limit connection count
- Apply message rate limits
- Set idle timeouts
- Validate message schemas

---

## 6.20 WebSockets vs HTTP

Traditional HTTP:

```text
Request
   |
Response
   |
Connection may close or be reused
```

WebSocket:

```text
One established connection
   |
Continuous bidirectional messages
```

WebSockets are useful when communication is frequent.

For occasional operations, normal HTTP may be simpler and more scalable.

---

## 6.21 WebSockets vs SSE

Use SSE when:

- Communication is server-to-client
- Messages are text
- Automatic reconnection is useful
- Simplicity matters

Use WebSockets when:

- Both sides send messages frequently
- Low latency is important
- Binary messages are needed
- Interactive communication is required

---

## 6.22 WebSocket Use Cases

- Chat
- Multiplayer games
- Collaborative editing
- Trading platforms
- Real-time bidding
- Live customer support
- Device control
- Real-time location tracking
- Interactive dashboards

---

## 6.23 WebSocket Interview Questions

- What are WebSockets?
- How does the WebSocket handshake work?
- What is HTTP status 101?
- What is the difference between `ws` and `wss`?
- How are WebSockets different from HTTP?
- What are WebSocket frames?
- What are ping and pong frames?
- How do you detect dead connections?
- How do you reconnect?
- How do you recover missed messages?
- How do you guarantee message ordering?
- Does WebSocket guarantee message delivery?
- How do acknowledgements work?
- How do you authenticate WebSockets?
- How do you handle token expiration?
- How do you authorize subscriptions?
- How do you scale WebSockets horizontally?
- Why is a shared pub/sub layer needed?
- What are sticky sessions?
- How do you handle backpressure?
- How do you protect against connection exhaustion?
- WebSockets vs SSE?
- WebSockets vs long polling?
- When should WebSockets not be used?

---

## 6.24 WebSocket Interview Readiness

- [ ] I can explain WebSockets in my own words.
- [ ] I understand the HTTP upgrade handshake.
- [ ] I know the difference between `ws` and `wss`.
- [ ] I understand WebSocket frame types.
- [ ] I can explain ping and pong.
- [ ] I can design a reconnection strategy.
- [ ] I understand ordering and sequence numbers.
- [ ] I understand delivery guarantees.
- [ ] I can explain acknowledgements and replay.
- [ ] I can explain authentication.
- [ ] I can explain token refresh.
- [ ] I can explain authorization for channels.
- [ ] I understand horizontal scaling.
- [ ] I can explain sticky sessions.
- [ ] I understand backpressure.
- [ ] I can discuss WebSocket security.
- [ ] I can compare WebSockets with SSE.
- [ ] I can design a production-ready real-time service.

---

# 7. WebRTC

## 7.1 What Is WebRTC?

WebRTC stands for **Web Real-Time Communication**.

It enables real-time communication between browsers and applications.

It supports:

- Audio
- Video
- Screen sharing
- Peer-to-peer data channels

Typical applications include:

- Video meetings
- Voice calls
- Online interviews
- Screen sharing
- Telemedicine
- Peer-to-peer file transfer

---

## 7.2 WebRTC Is More Than Peer-to-Peer

WebRTC attempts to establish direct peer-to-peer communication.

However, direct communication is not always possible because of:

- Firewalls
- NAT devices
- Corporate networks
- Symmetric NAT
- Security policies

When direct communication fails, media may be relayed through a TURN server.

---

## 7.3 WebRTC Components

A complete WebRTC system normally includes:

- Signaling server
- ICE
- STUN server
- TURN server
- Peer connection
- Media streams
- Data channels
- Codecs
- Security protocols

---

## 7.4 Signaling

WebRTC does not define a mandatory signaling protocol.

Applications must implement signaling using technologies such as:

- WebSockets
- REST
- SSE
- Messaging services

Signaling exchanges:

- Session descriptions
- ICE candidates
- Call invitations
- Accept or reject events
- Participant metadata

Example:

```text
Browser A
   |
   | Offer
   v
Signaling Server
   |
   | Offer
   v
Browser B
```

The signaling server coordinates connection setup but may not carry the actual media.

---

## 7.5 SDP

SDP stands for **Session Description Protocol**.

It describes communication capabilities such as:

- Audio codecs
- Video codecs
- Network addresses
- Media types
- Encryption details
- Transport information

Browser A creates an offer.

Browser B creates an answer.

```text
Browser A -- SDP Offer --> Browser B
Browser A <-- SDP Answer -- Browser B
```

---

## 7.6 ICE

ICE stands for **Interactive Connectivity Establishment**.

ICE identifies possible network paths between peers.

Each possible path is represented by an ICE candidate.

Candidate types include:

- Host candidate
- Server reflexive candidate
- Relay candidate

---

## 7.7 Host Candidate

A host candidate represents a local network address.

Example:

```text
192.168.1.10
```

This may work when both peers are on the same network but usually cannot be used directly over the public internet.

---

## 7.8 STUN

STUN stands for **Session Traversal Utilities for NAT**.

A STUN server helps a client discover its public-facing IP address and port.

```text
Browser
   |
   | What is my public address?
   v
STUN Server
   |
   | Public IP and port
   v
Browser
```

The discovered address becomes a server reflexive candidate.

STUN does not relay media.

---

## 7.9 TURN

TURN stands for **Traversal Using Relays around NAT**.

When direct peer-to-peer communication is not possible, the TURN server relays traffic.

```text
Peer A
   |
   v
TURN Server
   |
   v
Peer B
```

TURN is operationally expensive because audio, video or data passes through the relay.

Production WebRTC systems normally need TURN because direct connections cannot be guaranteed.

---

## 7.10 ICE Candidate Selection

Peers gather multiple candidates.

Example:

```text
Host candidate
Server reflexive candidate
Relay candidate
```

ICE tests candidate pairs and selects a working path.

Preference is generally toward an efficient direct path, but connectivity and policy determine the final result.

---

## 7.11 Trickle ICE

Without trickle ICE, the application waits until all candidates are discovered before sending them.

With trickle ICE, candidates are sent as they become available.

This can reduce call setup time.

```text
Candidate 1 discovered -> send immediately
Candidate 2 discovered -> send immediately
Candidate 3 discovered -> send immediately
```

---

## 7.12 Peer Connection

The browser uses:

```javascript
const peerConnection = new RTCPeerConnection(configuration);
```

Configuration may include STUN and TURN servers.

```javascript
const configuration = {
  iceServers: [
    {
      urls: "stun:stun.example.com:3478"
    },
    {
      urls: "turn:turn.example.com:3478",
      username: "temporary-user",
      credential: "temporary-password"
    }
  ]
};
```

TURN credentials should normally be temporary rather than permanently embedded in client code.

---

## 7.13 Accessing Camera and Microphone

```javascript
const stream = await navigator.mediaDevices.getUserMedia({
  video: true,
  audio: true
});
```

Tracks are added to the peer connection:

```javascript
stream.getTracks().forEach(track => {
  peerConnection.addTrack(track, stream);
});
```

The browser requests user permission before accessing devices.

---

## 7.14 Screen Sharing

```javascript
const screenStream =
  await navigator.mediaDevices.getDisplayMedia({
    video: true
  });
```

The screen-sharing track can be added or replace the existing video track.

Applications must handle:

- User stopping screen share
- Switching cameras
- Track replacement
- Permission changes
- Screen-share resolution

---

## 7.15 Codecs

WebRTC uses codecs to encode and decode media.

Audio codecs may include:

- Opus
- G.711

Video codecs may include:

- VP8
- VP9
- H.264
- AV1 in supported environments

Peers negotiate a compatible codec through SDP.

Codec selection affects:

- Quality
- CPU usage
- Bandwidth
- Device compatibility
- Licensing considerations

---

## 7.16 WebRTC Security

WebRTC media is encrypted.

Important technologies include:

- DTLS
- SRTP
- DTLS-SRTP
- Secure signaling through HTTPS or WSS

Media encryption does not remove the need to secure signaling.

An attacker who compromises signaling may interfere with connection establishment.

---

## 7.17 WebRTC Data Channels

WebRTC can send non-media data using data channels.

```javascript
const channel =
  peerConnection.createDataChannel("chat");
```

Use cases:

- Chat messages
- Game state
- File transfer
- Whiteboard updates
- Control signals

Data channels can be configured for:

- Reliable delivery
- Unreliable delivery
- Ordered delivery
- Unordered delivery

---

## 7.18 Mesh Architecture

In a mesh conference, each participant connects directly to every other participant.

For three users:

```text
A ⇄ B
A ⇄ C
B ⇄ C
```

Advantages:

- Simple architecture
- No central media server required
- Low server media cost when direct connections work

Limitations:

- Upload bandwidth grows rapidly
- CPU usage increases
- Poor fit for large meetings

For `n` participants, each participant may send media to approximately:

```text
n - 1 peers
```

---

## 7.19 SFU Architecture

SFU stands for **Selective Forwarding Unit**.

Each participant sends one media stream to the SFU.

The SFU forwards selected streams to other participants.

```text
Participant A
Participant B
Participant C
      |
      v
     SFU
      |
      v
Other Participants
```

Advantages:

- Better scaling than mesh
- Lower participant upload bandwidth
- Server does not normally need to mix all media
- Suitable for group meetings

Challenges:

- Server bandwidth
- Stream selection
- Simulcast management
- Operational complexity

---

## 7.20 MCU Architecture

MCU stands for **Multipoint Control Unit**.

The server receives media streams, mixes or transcodes them, and sends a combined stream.

Advantages:

- Client receives fewer streams
- Useful for lower-powered clients
- Centralized layout and recording

Limitations:

- High server CPU cost
- Additional latency
- Transcoding complexity
- Expensive infrastructure

---

## 7.21 Mesh vs SFU vs MCU

| Architecture | Client Upload | Server Processing | Scalability | Typical Use |
|---|---:|---:|---|---|
| Mesh | High as users increase | Low | Small calls | Small peer groups |
| SFU | Usually one main upload | Medium | Strong | Modern group conferencing |
| MCU | Usually one upload | High | Strong but expensive | Mixed-output conferencing |

---

## 7.22 Simulcast

With simulcast, a client sends multiple versions of the same video at different resolutions or bitrates.

Example:

```text
Low quality
Medium quality
High quality
```

The SFU selects an appropriate version for each receiver.

A mobile device may receive low quality.

A desktop with good bandwidth may receive high quality.

---

## 7.23 Network Adaptation

WebRTC adjusts to network conditions.

It may change:

- Bitrate
- Resolution
- Frame rate
- Packet retransmission behavior
- Selected video layer
- Codec configuration

The goal is to preserve call quality under changing network conditions.

---

## 7.24 Packet Loss, Jitter and Latency

### Packet loss

Some network packets do not arrive.

Effects:

- Audio gaps
- Video artifacts
- Frozen frames

### Jitter

Packet arrival times vary.

Jitter buffers help smooth playback but add latency.

### Latency

Time required for media to travel between participants.

High latency causes delayed conversation.

Real-time communication must balance:

- Quality
- Buffering
- Latency
- Retransmission

---

## 7.25 WebRTC Statistics

The browser provides statistics through:

```javascript
peerConnection.getStats();
```

Important metrics include:

- Round-trip time
- Packet loss
- Jitter
- Bitrate
- Frame rate
- Resolution
- Codec
- Candidate type
- Available bandwidth
- Packets sent and received

These metrics are essential for troubleshooting poor call quality.

---

## 7.26 WebRTC Connection Flow

```text
1. User A requests camera and microphone access.

2. User A creates RTCPeerConnection.

3. User A creates an SDP offer.

4. Offer is sent through the signaling server.

5. User B receives the offer.

6. User B creates an SDP answer.

7. Answer is returned through signaling.

8. Both peers gather ICE candidates.

9. ICE candidates are exchanged.

10. ICE connectivity checks run.

11. Direct or TURN-relayed path is selected.

12. Secure media begins.
```

---

## 7.27 WebRTC Failure Scenarios

Common failures include:

- Camera permission denied
- Microphone permission denied
- No compatible codec
- Signaling disconnection
- ICE negotiation failure
- TURN server unavailable
- Corporate firewall blocking UDP
- Poor bandwidth
- High packet loss
- Device switching failure
- Browser compatibility issue

Applications should provide meaningful user messages and fallback behavior.

---

## 7.28 WebRTC Interview Questions

- What is WebRTC?
- What is signaling?
- Does WebRTC define a signaling protocol?
- What is SDP?
- What is ICE?
- What is an ICE candidate?
- What is STUN?
- What is TURN?
- STUN vs TURN?
- Why is TURN required?
- What is trickle ICE?
- What is a peer connection?
- How is WebRTC secured?
- What are DTLS and SRTP?
- What are data channels?
- Can WebRTC transfer files?
- What is the difference between mesh, SFU and MCU?
- Why does mesh not scale well?
- What is simulcast?
- How does WebRTC handle poor networks?
- What metrics would you monitor?
- What happens when UDP is blocked?
- What is an ICE restart?
- How does screen sharing work?
- How do you debug a failed WebRTC call?

---

## 7.29 WebRTC Interview Readiness

- [ ] I can explain WebRTC in my own words.
- [ ] I understand signaling.
- [ ] I understand SDP offers and answers.
- [ ] I can explain ICE.
- [ ] I understand host, reflexive and relay candidates.
- [ ] I can explain STUN.
- [ ] I can explain TURN.
- [ ] I understand trickle ICE.
- [ ] I know how peer connections are created.
- [ ] I understand camera, microphone and screen capture.
- [ ] I understand codecs.
- [ ] I can explain WebRTC encryption.
- [ ] I understand data channels.
- [ ] I can compare mesh, SFU and MCU.
- [ ] I understand simulcast.
- [ ] I can explain packet loss, jitter and latency.
- [ ] I know which WebRTC metrics to monitor.
- [ ] I can explain common connection failures.
- [ ] I can describe the complete call setup flow.
- [ ] I can design a basic video-conferencing architecture.

---

# 8. Technology Comparison

## 8.1 Detailed Comparison

| Technology | Communication Style | Transport | Typical Format | Persistent Connection | Browser Friendly | Main Use |
|---|---|---|---|---:|---:|---|
| REST | Request-response | HTTP | JSON | No requirement | Yes | Standard APIs |
| GraphQL | Request-response and subscriptions | HTTP/WebSocket | JSON | Only for subscriptions | Yes | Flexible client queries |
| gRPC | RPC and streaming | HTTP/2 | Protocol Buffers | Often reused | Limited without gRPC-Web | Internal services |
| Polling | Repeated request-response | HTTP | JSON | No | Yes | Periodic status checks |
| Long Polling | Delayed request-response | HTTP | JSON | One request held temporarily | Yes | Near-real-time legacy updates |
| SSE | Server-to-client stream | HTTP | Text event stream | Yes | Yes | Notifications and live feeds |
| WebSockets | Bidirectional messaging | WebSocket | Text or binary | Yes | Yes | Interactive real-time systems |
| WebRTC | Peer media and data | UDP/TCP-based secure transports | Audio, video and binary data | Yes | Yes | Calls and peer communication |

---

## 8.2 Requirement-Based Selection

| Requirement | Recommended Choice |
|---|---|
| Public CRUD API | REST |
| Flexible frontend response shape | GraphQL |
| High-performance internal microservices | gRPC |
| Infrequent status checks | Polling |
| Legacy near-real-time updates | Long Polling |
| Server-only live updates | SSE |
| Bidirectional chat or collaboration | WebSockets |
| Voice, video or screen sharing | WebRTC |

---

## 8.3 One-Way vs Two-Way

### One-way request-response

```text
REST
GraphQL queries
Polling
Long polling
```

### Server-to-client streaming

```text
SSE
gRPC server streaming
```

### Bidirectional communication

```text
WebSockets
gRPC bidirectional streaming
WebRTC data channels
```

### Peer media communication

```text
WebRTC
```

---

## 8.4 How to Select a Technology

Ask these questions:

1. Is the communication request-response or real time?
2. Does the client need to send continuous messages?
3. Does only the server send updates?
4. Are audio or video involved?
5. Is the API public or internal?
6. Is browser compatibility required?
7. Are payload size and latency critical?
8. Is streaming required?
9. How many concurrent connections are expected?
10. What delivery guarantee is required?
11. Must messages be replayed?
12. Can the infrastructure support long-lived connections?
13. How will authentication work?
14. How will the system scale horizontally?
15. What observability is required?

---

# 9. Scenario-Based Interview Questions

## Scenario 1: Report Generation

A report takes five minutes to generate.

Recommended approach:

```text
POST /reports
        |
        v
202 Accepted
        |
        v
Background processing
        |
        v
Polling or SSE for status
```

Use polling when:

- Updates are infrequent
- A delay is acceptable
- Simplicity is important

Use SSE when:

- Progress should update continuously
- Many status changes are expected
- The browser remains open

---

## Scenario 2: Chat Application

Requirements:

- Users send messages
- Server sends messages
- Low latency
- Presence updates
- Typing indicators

Recommended:

```text
WebSockets
```

Additional components:

- Authentication
- Room authorization
- Pub/sub for horizontal scaling
- Message persistence
- Acknowledgements
- Reconnection and replay
- Presence tracking

---

## Scenario 3: Live Monitoring Dashboard

Requirements:

- Server sends metrics
- Client does not need frequent outbound messages
- Text or JSON updates

Recommended:

```text
SSE
```

WebSockets may be unnecessary unless the client must send continuous control messages.

---

## Scenario 4: Internal Microservice Communication

Requirements:

- High request volume
- Low latency
- Strong contracts
- Multiple programming languages
- Streaming may be required

Recommended:

```text
gRPC
```

REST may still be preferred when operational simplicity and debugging are more important than maximum efficiency.

---

## Scenario 5: Mobile Home Screen

Requirements:

- Multiple widgets
- Different clients need different fields
- Payload size matters

Recommended:

```text
GraphQL
```

Controls required:

- Query depth
- Query complexity
- Resolver batching
- Authorization
- Persisted queries

---

## Scenario 6: Payment Creation

Requirements:

- Avoid duplicate charges
- Client may retry after timeout

Recommended:

```text
REST POST with an idempotency key
```

Example:

```http
POST /payments
Idempotency-Key: payment-request-123
```

The server must persist the key and associated result.

---

## Scenario 7: Video Interview Platform

Requirements:

- Audio
- Video
- Screen sharing
- Low latency
- Multiple participants

Recommended:

```text
WebRTC
```

Supporting components:

- Signaling server
- STUN
- TURN
- SFU for group calls
- Call-quality monitoring
- Recording service if required

---

## Scenario 8: Order Status Tracking

Possible choices:

### Polling

Use when status changes infrequently.

### SSE

Use when the server should push updates.

### WebSockets

Use when users also interact continuously through the same real-time channel.

Do not choose WebSockets automatically when one-way updates are sufficient.

---

## Scenario 9: Collaborative Document Editing

Requirements:

- Multiple users edit simultaneously
- Bidirectional low-latency messages
- Presence indicators
- Conflict handling

Recommended:

```text
WebSockets
```

Additional design concerns:

- Operational transformation or CRDTs
- Versioning
- Ordering
- Reconnection
- Conflict resolution
- Presence
- Persistence

---

## Scenario 10: Stock Price Feed

Possible choices:

### SSE

Suitable when clients only receive price updates.

### WebSockets

Suitable when clients also submit real-time actions through the same connection.

For actual order placement, many systems still use secure HTTP or a carefully designed messaging protocol rather than assuming every action belongs on the price-feed connection.

---

# 10. Interview Readiness Tracker

## REST

- [ ] REST fundamentals
- [ ] REST constraints
- [ ] HTTP methods
- [ ] Safe methods
- [ ] Idempotency
- [ ] PUT vs PATCH
- [ ] Status codes
- [ ] 401 vs 403
- [ ] Pagination
- [ ] Caching
- [ ] ETags
- [ ] Versioning
- [ ] Error handling
- [ ] Authentication
- [ ] Authorization
- [ ] Rate limiting
- [ ] Long-running operations
- [ ] Security risks

## GraphQL

- [ ] Schema
- [ ] Types and nullability
- [ ] Queries
- [ ] Mutations
- [ ] Subscriptions
- [ ] Resolvers
- [ ] Variables
- [ ] Fragments
- [ ] N+1 problem
- [ ] DataLoader
- [ ] Partial responses
- [ ] Error handling
- [ ] Authorization
- [ ] Query complexity
- [ ] Query depth
- [ ] Caching
- [ ] Persisted queries
- [ ] Schema evolution
- [ ] Federation

## gRPC

- [ ] Protocol Buffers
- [ ] Code generation
- [ ] Unary RPC
- [ ] Server streaming
- [ ] Client streaming
- [ ] Bidirectional streaming
- [ ] HTTP/2
- [ ] Deadlines
- [ ] Cancellation
- [ ] Metadata
- [ ] Interceptors
- [ ] Status codes
- [ ] TLS and mTLS
- [ ] Load balancing
- [ ] Retries
- [ ] Backward compatibility
- [ ] gRPC-Web

## Polling

- [ ] Fixed polling
- [ ] Polling interval
- [ ] Load calculation
- [ ] Conditional polling
- [ ] ETags
- [ ] Adaptive polling
- [ ] Exponential backoff
- [ ] Jitter
- [ ] Long polling
- [ ] Overlapping requests
- [ ] Retry storms
- [ ] When polling is appropriate

## SSE

- [ ] EventSource
- [ ] `text/event-stream`
- [ ] Event format
- [ ] Named events
- [ ] Reconnection
- [ ] `Last-Event-ID`
- [ ] Heartbeats
- [ ] Authentication
- [ ] Horizontal scaling
- [ ] Pub/sub
- [ ] Backpressure
- [ ] Proxy behavior
- [ ] SSE vs polling
- [ ] SSE vs WebSockets

## WebSockets

- [ ] HTTP upgrade handshake
- [ ] Status 101
- [ ] `ws` vs `wss`
- [ ] Frames
- [ ] Ping and pong
- [ ] Reconnection
- [ ] Sequence numbers
- [ ] Delivery guarantees
- [ ] Acknowledgements
- [ ] Authentication
- [ ] Token expiration
- [ ] Authorization
- [ ] Horizontal scaling
- [ ] Shared pub/sub
- [ ] Sticky sessions
- [ ] Connection registry
- [ ] Backpressure
- [ ] Security
- [ ] WebSockets vs SSE

## WebRTC

- [ ] Signaling
- [ ] SDP
- [ ] ICE
- [ ] ICE candidates
- [ ] STUN
- [ ] TURN
- [ ] Trickle ICE
- [ ] Peer connection
- [ ] Camera and microphone
- [ ] Screen sharing
- [ ] Codecs
- [ ] DTLS and SRTP
- [ ] Data channels
- [ ] Mesh
- [ ] SFU
- [ ] MCU
- [ ] Simulcast
- [ ] Network adaptation
- [ ] Packet loss
- [ ] Jitter
- [ ] Latency
- [ ] Statistics
- [ ] Connection failure debugging

---

# Final Interview Answer

> I select the communication mechanism based on the interaction pattern, consumers, performance requirements and operational complexity. REST is my default for standard resource-oriented APIs because it is simple, stateless and widely supported. GraphQL is useful when frontend clients need flexible response shapes, but it requires controls for query complexity, authorization and N+1 problems. gRPC is a strong choice for high-performance internal service communication because it uses HTTP/2, Protocol Buffers and strongly typed contracts. Polling is acceptable for infrequent status checks, while SSE is better for one-way server push such as notifications and dashboards. WebSockets are appropriate when both client and server exchange frequent real-time messages. WebRTC is specifically designed for low-latency audio, video, screen sharing and peer data communication. I avoid choosing the most advanced option automatically because each technology introduces different scaling, security, observability and operational trade-offs.
