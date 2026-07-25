# Senior Software Engineering Notes

A structured collection of software engineering notes covering API design, Java, Spring Boot, microservices, AWS, databases, system design, DevOps, and interview preparation.

This repository is intended to serve as:

* A personal engineering knowledge base
* A quick revision guide for technical interviews
* A reference for backend and full-stack architecture concepts
* A practical learning resource for senior software engineers

---

## Repository Structure

```text
senior-software-engineering-notes/
│
├── README.md
│
├── API Design/
│   ├── ApplicationLayerProtocols.md
│   └── ApplicationCommunicationTechnologies-DetailedInterviewGuide.md
│
├── Java/
├── Spring Boot/
├── Microservices/
├── AWS/
├── Databases/
├── System Design/
├── Docker-Kubernetes/
├── Design Patterns/
└── Interview Notes/
```

---

# API Design

The API Design section covers communication styles, protocols, API patterns, scalability considerations, security, and interview preparation.

| Topic                                                                                                                                            | Description                                                                                                                                                                                    |
| ------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Application Communication Protocols and Patterns](API%20Design/ApplicationLayerProtocols.md)                                                    | High-level overview and evolution of REST, GraphQL, gRPC, Polling, Long Polling, Server-Sent Events, WebSockets, and WebRTC                                                                    |
| [Application Communication Technologies — Detailed Interview Guide](API%20Design/ApplicationCommunicationTechnologies-DetailedInterviewGuide.md) | Detailed explanation of each communication technology, including internal working, security, scalability, failure handling, practical scenarios, interview questions, and readiness checklists |

---

## Application Communication Protocols and Patterns

The [Application Communication Protocols and Patterns](API%20Design/ApplicationLayerProtocols.md) document provides a concise overview of application communication technologies.

It explains two major evolution tracks.

### API Communication

```text
SOAP
   |
   v
REST
   |
   +----------------+
   |                |
   v                v
GraphQL            gRPC
```

### Real-Time Communication

```text
Polling
   |
   v
Long Polling
   |
   v
Server-Sent Events
   |
   v
WebSockets
   |
   v
WebRTC
```

Topics covered:

* REST
* GraphQL
* gRPC
* Polling
* Long Polling
* Server-Sent Events
* WebSockets
* WebRTC
* Technology comparison
* When to choose each communication mechanism

This document is useful for quickly understanding what problem each technology solves.

---

## Detailed Application Communication Guide

The [Application Communication Technologies — Detailed Interview Guide](API%20Design/ApplicationCommunicationTechnologies-DetailedInterviewGuide.md) provides an in-depth explanation of every communication technology.

It includes:

* Internal working
* Architecture flow
* HTTP methods and status codes
* REST constraints
* Idempotency
* Pagination and caching
* GraphQL schemas and resolvers
* GraphQL N+1 problem
* Protocol Buffers
* gRPC streaming models
* Polling optimization
* Exponential backoff and jitter
* SSE event format
* WebSocket handshake
* WebSocket scaling and delivery guarantees
* WebRTC signaling
* STUN, TURN, ICE, SDP
* Mesh, SFU, and MCU architectures
* Security considerations
* Performance considerations
* Failure handling
* Scenario-based interview questions
* Interview readiness checklists

This guide is intended for deeper learning and senior-level interview preparation.

---

# Planned Topics

## REST API Design

* Resource naming conventions
* HTTP methods
* HTTP status codes
* Idempotency
* Pagination
* Filtering
* Sorting
* API versioning
* Error response design
* Content negotiation
* Caching
* ETags
* Rate limiting
* API security
* OpenAPI and Swagger

---

## Microservices

* Microservices architecture
* Service decomposition
* Synchronous communication
* Asynchronous communication
* Event-driven architecture
* Saga pattern
* Compensating transactions
* Circuit breaker
* Retry and timeout handling
* Service discovery
* Distributed tracing
* Transactional outbox
* Dead-letter queues
* Idempotent consumers
* Eventual consistency

---

## Java

* Java 8 features
* Java 11 features
* Java 17 features
* Java 21 features
* Collections framework
* HashMap internals
* Multithreading
* CompletableFuture
* Virtual threads
* Java Memory Model
* Garbage collection
* JVM internals
* Exception handling
* Functional interfaces
* Streams
* Optional
* Records
* Sealed classes

---

## Spring Boot

* Spring Boot fundamentals
* Dependency injection
* Spring annotations
* Spring MVC
* REST API development
* Exception handling
* Validation
* Spring Data JPA
* Hibernate
* Spring Security
* OAuth 2.0
* OpenID Connect
* JWT validation
* Actuator
* Configuration management
* Testing with JUnit and Mockito
* Resilience4j
* Spring Boot on AWS

---

## AWS

* Application Load Balancer
* Elastic Container Service
* Elastic Container Registry
* AWS Lambda
* Amazon SQS
* Dead-letter queues
* Amazon SNS
* Amazon S3
* Amazon RDS
* Amazon CloudWatch
* AWS IAM
* VPC
* Route 53
* Auto Scaling
* Cloud-native monitoring
* Infrastructure as code

---

## Databases

* PostgreSQL
* Oracle
* DB2
* Database normalization
* Indexes
* Transactions
* ACID properties
* Isolation levels
* Optimistic locking
* Pessimistic locking
* Query optimization
* Connection pooling
* Database partitioning
* Replication
* Sharding
* NoSQL databases
* SQL vs NoSQL

---

## System Design

* URL shortener
* Notification system
* Chat application
* Payment system
* Order management system
* Rate limiter
* Distributed cache
* File storage system
* Logging platform
* Monitoring system
* Video streaming platform
* Search autocomplete
* API gateway design
* Event-driven systems

---

## Docker and Kubernetes

* Docker images
* Containers
* Dockerfile
* RUN, CMD, and ENTRYPOINT
* Docker networking
* Docker volumes
* Multi-stage builds
* Kubernetes architecture
* Pods
* Deployments
* Services
* ConfigMaps
* Secrets
* Ingress
* Horizontal Pod Autoscaler
* Helm
* Container security
* CI/CD deployment patterns

---

## Design Patterns

* SOLID principles
* Singleton
* Factory
* Abstract Factory
* Builder
* Strategy
* Observer
* Adapter
* Facade
* Decorator
* Template Method
* Chain of Responsibility
* Command
* Proxy
* Repository pattern

---

# Target Audience

This repository is useful for:

* Senior Software Engineers
* Java Developers
* Spring Boot Developers
* Backend Engineers
* Full-Stack Engineers
* Technical Leads
* Solution Architects
* Developers preparing for system design interviews
* Developers preparing for senior engineering interviews

---

# Learning Approach

The repository focuses on understanding:

```text
What problem does the technology solve?

How does it work internally?

When should it be used?

When should it not be used?

How does it scale?

How is it secured?

What can fail?

What is commonly asked in interviews?
```

The goal is not to memorize definitions.

The goal is to understand the engineering trade-offs and explain them using practical examples.

---

# Suggested Reading Order

1. [Application Communication Protocols and Patterns](API%20Design/ApplicationLayerProtocols.md)
2. [Application Communication Technologies — Detailed Interview Guide](API%20Design/ApplicationCommunicationTechnologies-DetailedInterviewGuide.md)
3. REST API Design
4. Microservices Communication
5. Event-Driven Architecture
6. Resilience Patterns
7. Distributed System Design
8. Security and Observability

---

# Repository Status

This repository is actively being expanded.

```text
API Design              In Progress
Java                   Planned
Spring Boot            Planned
Microservices          Planned
AWS                    Planned
Databases              Planned
System Design          Planned
Docker and Kubernetes  Planned
Design Patterns        Planned
Interview Notes        Planned
```

---

# Contribution and Usage

These notes are maintained as a personal learning and interview preparation resource.

The examples are intentionally simplified where necessary so that the underlying concepts remain easy to understand.

Before applying any design in production, consider:

* Business requirements
* Expected traffic
* Security requirements
* Availability requirements
* Infrastructure limitations
* Team expertise
* Operational complexity
* Cost
* Monitoring
* Failure recovery

---

# Disclaimer

The technologies documented in this repository should be selected based on the problem being solved.

There is no single communication protocol, framework, database, or architecture that is best for every application.

Good software architecture is based on informed trade-offs rather than technology trends.
