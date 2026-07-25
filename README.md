# Senior Software Engineering Notes

A structured collection of notes covering API design, networking, Java, Spring Boot, microservices, AWS, databases, system design, DevOps, and senior-level interview preparation.

This repository serves as:

- A personal engineering knowledge base
- A technical interview revision guide
- A reference for backend and full-stack architecture concepts
- A practical learning resource for senior software engineers

---

# Repository Structure

```text
senior-software-engineering-notes/
│
├── README.md
│
├── API Design/
│   ├── ApplicationLayerProtocols.md
│   └── ApplicationCommunicationTechnologies-DetailedInterviewGuide.md
│
├── Networking/
│   ├── TCPvsUDP-Short.md
│   └── TCPvsUDP-Detail.md
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

# Quick Navigation

| Category | Guide | Level | Description |
|---|---|---|---|
| API Design | [Application Communication Protocols and Patterns](API%20Design/ApplicationLayerProtocols.md) | Overview | Concise explanation of REST, GraphQL, gRPC, Polling, Long Polling, SSE, WebSockets, and WebRTC |
| API Design | [Application Communication Technologies — Detailed Interview Guide](API%20Design/ApplicationCommunicationTechnologies-DetailedInterviewGuide.md) | Detailed | Internal behavior, scalability, security, failure handling, scenarios, and interview questions |
| Networking | [TCP vs UDP](Networking/TCPvsUDP.md) | Overview | Concise explanation of the transport layer, TCP, UDP, key differences, use cases, and common interview questions |
| Networking | [TCP vs UDP — Detailed Interview Guide](Networking/TCPvsUDP-DetailedInterviewGuide.md) | Detailed | Handshakes, reliability, flow control, congestion control, datagrams, QUIC, troubleshooting, and interview preparation |

---

# API Design

## Application Communication Protocols and Patterns

[Read the overview](API%20Design/ApplicationLayerProtocols.md)

Covers:

- REST
- GraphQL
- gRPC
- Polling
- Long Polling
- Server-Sent Events
- WebSockets
- WebRTC
- Communication-pattern selection

## Detailed Application Communication Guide

[Read the detailed interview guide](API%20Design/ApplicationCommunicationTechnologies-DetailedInterviewGuide.md)

Covers:

- REST constraints and API design
- HTTP methods and status codes
- Idempotency
- Pagination and caching
- GraphQL schemas and resolvers
- N+1 queries and DataLoader
- gRPC and Protocol Buffers
- Streaming models
- Polling and long polling
- SSE event delivery
- WebSocket scaling
- WebRTC signaling, STUN, TURN, ICE, SFU, and MCU
- Security, failure handling, and interview questions

---

# Networking

## TCP vs UDP Overview

[Read the concise TCP vs UDP guide](Networking/TCPvsUDP.md)

Covers:

- Transport-layer basics
- IP addresses, ports, and sockets
- TCP connection establishment
- TCP reliability and ordering
- TCP flow and congestion control
- UDP datagrams
- Real-time communication
- QUIC and HTTP/3
- TCP vs UDP comparison
- Common scenarios and interview questions

## TCP vs UDP Detailed Interview Guide

[Read the detailed TCP vs UDP guide](Networking/TCPvsUDP-DetailedInterviewGuide.md)

Covers:

- Transport-layer internals
- TCP segment fields
- Three-way handshake
- Sequence and acknowledgement numbers
- Retransmission and SACK
- Sliding windows
- Flow control
- Congestion control
- TCP termination and `TIME_WAIT`
- UDP headers and datagrams
- Application-level reliability
- Broadcast and multicast
- Jitter buffers and forward error correction
- QUIC and HTTP/3
- Production troubleshooting and observability
- Detailed interview questions and readiness checklist

---

# Planned Networking Topics

```text
Networking/
│
├── TCPvsUDP.md
├── TCPvsUDP-DetailedInterviewGuide.md
├── OSIAndTCPIPModels.md
├── IPAddressingAndSubnetting.md
├── DNSAndDHCP.md
├── HTTPAndHTTPS.md
├── TLSAndCertificates.md
├── RoutingNATAndFirewalls.md
├── LoadBalancersAndProxies.md
└── NetworkTroubleshooting.md
```

Future topics:

- OSI and TCP/IP models
- IPv4 and IPv6
- Subnetting
- DNS and DHCP
- ARP
- NAT
- Routing
- HTTP and HTTPS
- TLS handshakes and certificates
- Firewalls and VPNs
- Load balancers and proxies
- Content delivery networks
- Network troubleshooting

---

# Planned Engineering Topics

## Java

- Java 8, 11, 17, and 21
- Collections and HashMap internals
- Multithreading and concurrency
- CompletableFuture
- Virtual threads
- Java Memory Model
- JVM and garbage collection

## Spring Boot

- Dependency injection
- Spring MVC
- REST APIs
- Validation and exception handling
- Spring Data JPA
- Spring Security
- OAuth 2.0, OIDC, and JWT
- Actuator
- Resilience4j
- Testing

## Microservices

- Service decomposition
- Synchronous and asynchronous communication
- Event-driven architecture
- Saga pattern
- Transactional outbox
- Circuit breaker
- Retry and timeout handling
- Distributed tracing
- Eventual consistency

## AWS

- ALB
- ECS and ECR
- Lambda
- SQS and SNS
- RDS
- CloudWatch
- IAM
- VPC
- Route 53
- Auto Scaling

## Databases

- PostgreSQL, Oracle, and DB2
- Indexing
- Transactions and isolation levels
- Optimistic and pessimistic locking
- Query optimization
- Partitioning
- Replication
- Sharding
- SQL vs NoSQL

## System Design

- URL shortener
- Notification system
- Chat system
- Payment system
- Rate limiter
- Distributed cache
- File storage
- Monitoring and logging
- Video streaming
- Search autocomplete

## Docker and Kubernetes

- Images and containers
- Dockerfile
- RUN, CMD, and ENTRYPOINT
- Networking and volumes
- Multi-stage builds
- Pods, deployments, and services
- ConfigMaps and secrets
- Ingress
- Autoscaling
- Helm
- Container security

---

# Learning Approach

Each guide aims to answer:

```text
What problem does this technology solve?

How does it work internally?

When should it be used?

When should it not be used?

How does it scale?

How is it secured?

What can fail?

What is commonly asked in interviews?
```

The goal is to understand engineering trade-offs rather than memorize definitions.

---

# Suggested Reading Order

1. [Application Communication Protocols and Patterns](API%20Design/ApplicationLayerProtocols.md)
2. [Application Communication Technologies — Detailed Interview Guide](API%20Design/ApplicationCommunicationTechnologies-DetailedInterviewGuide.md)
3. [TCP vs UDP](Networking/TCPvsUDP.md)
4. [TCP vs UDP — Detailed Interview Guide](Networking/TCPvsUDP-DetailedInterviewGuide.md)
5. OSI and TCP/IP Models
6. IP Addressing and Subnetting
7. DNS and DHCP
8. HTTP, HTTPS, and TLS
9. Microservices Communication
10. Distributed Systems and Resilience

---

# Repository Status

```text
API Design              In Progress
Networking              In Progress
Java                    Planned
Spring Boot             Planned
Microservices           Planned
AWS                     Planned
Databases               Planned
System Design           Planned
Docker and Kubernetes   Planned
Design Patterns         Planned
Interview Notes         Planned
```

---

# Disclaimer

The technologies in this repository should be selected based on the problem being solved.

There is no single protocol, framework, database, or architecture that is best for every system.

Good engineering decisions are based on informed trade-offs involving reliability, latency, scalability, security, complexity, cost, and team expertise.
