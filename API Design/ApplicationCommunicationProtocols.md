# Application Communication Protocols & Patterns

This document explains the evolution of modern application communication mechanisms and when to use each one.

Instead of memorizing technologies, understand **what problem each one solves**. Most of these technologies were introduced to overcome limitations of previous approaches.

---

# Two Evolution Tracks

Communication technologies evolved in **two different directions**.

## 1. API Communication Evolution

This focuses on how applications exchange request-response data.

```text
SOAP
   │
   ▼
REST
   │
   ├──────────────┐
   ▼              ▼
GraphQL          gRPC
```

---

## 2. Real-Time Communication Evolution

This focuses on delivering live updates and real-time communication.

```text
Polling
   │
   ▼
Server-Sent Events (SSE)
   │
   ▼
WebSockets
   │
   ▼
WebRTC
```

---

# API Communication

---

# 1. REST

## Purpose

REST (Representational State Transfer) is the standard architectural style for building web APIs.

It uses HTTP methods such as:

- GET
- POST
- PUT
- DELETE

Data is generally exchanged using JSON.

---

## Example

```http
GET /customers/123
```

Response

```json
{
  "id": 123,
  "name": "John"
}
```

---

## Advantages

- Simple
- Stateless
- HTTP based
- Easy caching
- Easy scaling
- Widely supported

---

## Practical Examples

- Banking APIs
- E-commerce applications
- Mobile applications
- CRUD systems
- Public APIs

---

## Problem with REST

Suppose a React page needs

- Customer
- Orders
- Payments

REST requires

```text
GET /customer/1

GET /orders/1

GET /payments/1
```

Multiple network calls.

Or

```text
GET /customerDetails
```

returns

```text
200 fields
```

while the UI only needs

```text
5 fields
```

Problems

- Over-fetching
- Under-fetching
- Multiple API calls

---

# 2. GraphQL

## Why GraphQL?

GraphQL solves REST's over-fetching and under-fetching problem.

The client decides what data it needs instead of the server deciding.

---

## Example

```graphql
query {
  customer {
    name
    orders {
      id
      amount
    }
  }
}
```

Response

```json
{
  "name": "John",
  "orders": [
    {
      "id": 101,
      "amount": 250
    }
  ]
}
```

Only requested fields are returned.

---

## Advantages

- Single endpoint
- Flexible responses
- Reduces API calls
- Eliminates over-fetching
- Great for complex UIs

---

## Practical Examples

- Netflix
- Facebook
- GitHub API
- React dashboards
- Mobile applications

---

## Problem

GraphQL still follows

```text
Request

↓

Response

↓

Connection closes
```

It is not optimized for

- High-performance service communication
- Binary serialization
- Large-scale internal microservice communication

---

# 3. gRPC

## Why gRPC?

gRPC solves REST/GraphQL performance limitations.

Instead of JSON, it uses

- Protocol Buffers (Binary Serialization)

Runs on

- HTTP/2

---

## Advantages

- Very fast
- Smaller payloads
- Streaming support
- Strong contracts
- Efficient internal communication

---

## Practical Examples

```text
Order Service

↓

Inventory Service

↓

Pricing Service
```

Ideal when thousands of internal service calls happen every second.

---

## Problem

- Difficult to debug (binary protocol)
- Browser support is limited
- Primarily designed for backend-to-backend communication

---

# Real-Time Communication

---

# 4. Polling

## Purpose

Polling is the simplest way to check whether new information is available.

The client repeatedly sends requests at fixed intervals.

---

## Example

```text
Browser

↓

GET /notifications

↓

No updates

(wait 5 seconds)

↓

GET /notifications

↓

No updates

(wait 5 seconds)

↓

GET /notifications

↓

New notification
```

---

## Advantages

- Very easy to implement
- Works with every HTTP server
- No persistent connection

---

## Practical Examples

- Order status page
- Report generation status
- Legacy dashboards
- Admin portals

---

## Problem

Imagine

- 10,000 users
- Polling every 5 seconds

```text
10,000 Users

↓

120,000 Requests / Minute

↓

Most responses

"No Updates"
```

This wastes

- CPU
- Network bandwidth
- Server resources

The server keeps responding even though nothing has changed.

---

## Solution

Instead of repeatedly asking,

the server should notify the client only when data changes.

This led to **Server-Sent Events (SSE).**

---

# 5. Server-Sent Events (SSE)

## Why SSE?

SSE eliminates unnecessary polling.

The client opens one HTTP connection.

Whenever something changes,

the server pushes the update.

---

## Communication

```text
Server

↓

Client
```

One-way communication.

---

## Advantages

- Lightweight
- HTTP based
- Automatic reconnect
- Simpler than WebSockets

---

## Practical Examples

- Notifications
- Monitoring dashboards
- Live sports scores
- News feeds
- Stock price updates (read-only)

---

## Problem

SSE only supports

```text
Server

↓

Client
```

It cannot support

- Chat
- Multiplayer games
- Collaborative editing

Need two-way communication.

---

# 6. WebSockets

## Why WebSockets?

WebSockets provide

**Bidirectional Communication**

The connection remains open,

and both client and server can send data anytime.

---

## Communication

```text
Client
⇅
Server
```

---

## Advantages

- Very low latency
- Persistent connection
- Bidirectional communication
- Real-time updates

---

## Practical Examples

- WhatsApp
- Slack
- Google Docs collaboration
- Multiplayer games
- Trading platforms

---

## Problem

Maintaining one TCP connection per client consumes resources.

Sometimes the requirement isn't just messaging.

It involves

- Audio
- Video
- Screen sharing

Need direct browser-to-browser communication.

---

# 7. WebRTC

## Why WebRTC?

WebRTC is designed for

- Audio
- Video
- Screen sharing
- Peer-to-peer communication

---

## Communication

Instead of

```text
Browser

↓

Server

↓

Browser
```

Communication becomes

```text
Browser
⇅
Browser
```

The server helps establish the connection (signaling), after which media is exchanged directly between peers whenever possible.

---

## Advantages

- Extremely low latency
- Peer-to-peer
- Voice
- Video
- Screen sharing
- Data channels

---

## Practical Examples

- Google Meet
- Zoom
- Microsoft Teams
- Online interviews
- Telemedicine

---

# Quick Comparison

| Technology | Communication | Primary Purpose | Best Use Cases |
|------------|--------------|-----------------|----------------|
| REST | Request → Response | Standard APIs | CRUD APIs, Mobile Apps, Public APIs |
| GraphQL | Request → Response | Flexible data fetching | React, Angular, Mobile Apps |
| gRPC | Request → Response / Streaming | High-performance service communication | Internal Microservices |
| Polling | Client repeatedly asks | Periodic status checking | Legacy systems, Status pages |
| SSE | Server → Client | Server push | Notifications, Dashboards |
| WebSockets | Client ⇄ Server | Real-time bidirectional communication | Chat, Gaming, Collaboration |
| WebRTC | Peer ⇄ Peer | Audio/Video communication | Video Calls, Screen Sharing |

---

# Which One Should I Choose?

## Use REST when

- Building standard CRUD APIs
- Exposing public APIs
- Supporting web/mobile applications
- Simplicity is the priority

---

## Use GraphQL when

- Frontend needs different fields
- Reduce over-fetching
- Multiple clients consume the same API

---

## Use gRPC when

- Communication is between microservices
- Performance is critical
- Low latency is required
- Strong contracts are important

---

## Use Polling when

- Updates are infrequent
- Simplicity matters
- Legacy systems
- Status tracking pages

---

## Use SSE when

- Only the server sends updates
- Live dashboards
- Notifications
- Monitoring systems

---

## Use WebSockets when

- Both client and server continuously exchange messages
- Chat applications
- Multiplayer games
- Collaborative editing

---

## Use WebRTC when

- Video conferencing
- Voice calls
- Screen sharing
- Browser-to-browser communication

---

# Senior Interview Summary

> I choose the communication mechanism based on the business requirement rather than following a trend. REST is my default choice for standard CRUD operations and public APIs because it is simple, stateless, and widely supported. When the frontend needs flexible data fetching, GraphQL helps eliminate over-fetching and reduce multiple API calls. For high-performance internal microservice communication, gRPC is preferred because HTTP/2 and Protocol Buffers provide lower latency and smaller payloads. For real-time updates, Polling is the simplest approach but wastes resources due to repeated requests. SSE improves this by allowing the server to push updates over a single connection. When both client and server need to communicate in real time, WebSockets provide full-duplex communication. Finally, for browser-based audio, video, and screen sharing, WebRTC enables low-latency peer-to-peer media communication.