# REST vs GraphQL vs gRPC vs WebSockets vs SSE vs WebRTC

One of the easiest ways to understand these communication technologies is to learn **why each one was introduced**. Every new technology was designed to solve a limitation of the previous one.

---

# Evolution

```text
SOAP
   ↓
REST
   ↓ (Over-fetching & Multiple API Calls)
GraphQL
   ↓ (Performance & Internal Communication)
gRPC
   ↓ (Still Request-Response)
WebSockets
   ↓ (Too Heavy for One-Way Updates)
SSE
   ↓ (Cannot Handle Peer-to-Peer Audio/Video)
WebRTC
```

---

# 1. REST

## Purpose

REST (Representational State Transfer) is the most common architectural style for building web APIs.

It uses standard HTTP methods:

- GET
- POST
- PUT
- DELETE

Data is usually exchanged in JSON format.

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
- Easy to understand
- Easy caching
- Highly scalable
- Supported everywhere

---

## Practical Examples

- E-commerce applications
- Banking APIs
- Mobile applications
- CRUD applications
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
- Multiple round trips

---

# 2. GraphQL

## Why GraphQL?

GraphQL solves REST's over-fetching and under-fetching problem.

Instead of the server deciding what data to return,

the client decides.

---

## Example

Request

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

Only requested data is returned.

---

## Advantages

- Single endpoint
- Client controls response
- Reduces API calls
- Eliminates over-fetching
- Flexible for frontend teams

---

## Practical Examples

- Netflix
- Facebook
- GitHub API
- React dashboards
- Mobile applications

---

## Problem with GraphQL

Still follows

```text
Request

↓

Response

↓

Connection closes
```

Not ideal for

- High-performance service communication
- Binary payloads
- Streaming millions of requests

---

# 3. gRPC

## Why gRPC?

gRPC solves REST/GraphQL performance limitations.

Instead of JSON,

it uses

**Protocol Buffers (Binary Serialization).**

Runs over

- HTTP/2

---

## Advantages

- Extremely fast
- Small payloads
- HTTP/2
- Multiplexing
- Supports streaming
- Strong contracts (.proto files)

---

## Practical Examples

Microservices

```text
Order Service

↓

Inventory Service

↓

Pricing Service
```

Thousands of internal service calls per second.

---

## Problem

- Browsers don't naturally support it
- Binary data is difficult to debug
- Mostly used for backend communication
- Still request-response

---

# 4. WebSockets

## Why WebSockets?

REST, GraphQL and gRPC are primarily request-response.

Sometimes both client and server need to communicate continuously.

WebSockets provide

**Full Duplex Communication**

---

## Connection

```text
Client
⇅
Server
```

Both sides can send messages anytime.

---

## Advantages

- Real-time
- Persistent connection
- Very low latency
- Bidirectional communication

---

## Practical Examples

- WhatsApp
- Slack
- Online gaming
- Stock trading
- Google Docs collaboration

---

## Problem

Every client maintains an open TCP connection.

Imagine

```text
1 Million Users

↓

1 Million Open Connections
```

Consumes server resources.

Sometimes clients never send anything.

They only receive updates.

---

# 5. SSE (Server-Sent Events)

## Why SSE?

If communication is only

Server → Client

WebSockets become unnecessary.

SSE provides

One-way streaming.

---

## Connection

```text
Server

↓

Client
```

Connection remains open.

Server pushes updates whenever needed.

---

## Advantages

- Simpler than WebSockets
- HTTP based
- Automatic reconnect
- Lightweight
- Easy to implement

---

## Practical Examples

- Notification systems
- Live dashboards
- Monitoring
- News feeds
- Live sports scores

---

## Problem

Cannot support

- Client-to-server streaming
- Audio
- Video
- Peer communication

Need peer-to-peer media.

---

# 6. WebRTC

## Why WebRTC?

WebRTC is designed specifically for

- Audio
- Video
- Screen sharing
- Peer-to-peer communication

---

## Connection

Instead of

```text
Browser

↓

Server

↓

Browser
```

WebRTC establishes

```text
Browser
⇅
Browser
```

The server mainly helps establish the connection (signaling). After that, media is exchanged directly between peers whenever possible.

---

## Advantages

- Very low latency
- Peer-to-peer
- Video
- Voice
- Screen sharing
- Data channels

---

## Practical Examples

- Google Meet
- Zoom
- Microsoft Teams
- Telemedicine
- Online interviews

---

# Complete Evolution Summary

```text
SOAP

↓

REST
(Simple Web APIs)

↓

Problem
Over-fetching
Multiple API Calls

↓

GraphQL
(Client Controls Data)

↓

Problem
Performance

↓

gRPC
(Fast Binary Communication)

↓

Problem
Still Request-Response

↓

WebSockets
(Real-Time Bidirectional Communication)

↓

Problem
Too Heavy When Only Server Sends Updates

↓

SSE
(One-Way Server Push)

↓

Problem
Cannot Handle Audio/Video

↓

WebRTC
(Peer-to-Peer Media Communication)
```

---

# Comparison Table

| Technology | Main Purpose | Solves Which Problem | Best Use Cases |
|------------|--------------|----------------------|----------------|
| REST | Standard Web APIs | Simpler than SOAP | CRUD APIs, Mobile Apps, Public APIs |
| GraphQL | Client-controlled data fetching | REST over-fetching & under-fetching | React, Angular, Mobile Apps |
| gRPC | High-performance communication | JSON performance overhead | Internal Microservices |
| WebSockets | Full-duplex communication | Request-response limitation | Chat, Gaming, Trading |
| SSE | Server-to-client updates | Lightweight alternative to WebSockets | Notifications, Dashboards |
| WebRTC | Peer-to-peer media | Audio/video communication | Video Calls, Screen Sharing |

---

# When Should I Use Which?

## REST

Use when

- Standard CRUD APIs
- Public APIs
- Mobile applications
- External integrations

Example

```
Amazon Product API
```

---

## GraphQL

Use when

- Frontend needs flexible data
- Different clients need different fields
- Reduce API calls

Example

```
Netflix Home Screen
```

---

## gRPC

Use when

- Backend to Backend communication
- Internal microservices
- Low latency
- High throughput

Example

```
Order Service
↓

Inventory Service
```

---

## WebSockets

Use when

- Chat
- Gaming
- Stock Market
- Collaborative editing

Example

```
WhatsApp
```

---

## SSE

Use when

- Live notifications
- Monitoring dashboards
- News feeds
- Sports score updates

Example

```
Server continuously pushes CPU usage updates
```

---

## WebRTC

Use when

- Video conferencing
- Voice calling
- Screen sharing
- Browser peer-to-peer communication

Example

```
Google Meet
```

---

# Senior Interview Answer

> I choose the communication technology based on the business requirement rather than following a trend. REST is my default choice for standard CRUD operations and public APIs because it is simple, stateless, and widely supported. When the frontend requires flexible data fetching and wants to avoid over-fetching or multiple API calls, GraphQL is a better fit. For high-performance internal microservice communication, I prefer gRPC because HTTP/2 and Protocol Buffers provide lower latency and smaller payloads. For real-time bidirectional communication like chat or collaborative applications, WebSockets are the right choice. If the requirement is only server-to-client updates, such as notifications or dashboards, SSE is simpler and more efficient. For browser-based audio, video, and screen sharing, WebRTC is the best choice because it provides peer-to-peer real-time media communication.
