# TCP vs UDP

A concise guide to the two primary transport-layer protocols: **TCP** and **UDP**.

This document focuses on:

- Where TCP and UDP fit in networking
- How each protocol works
- Their main differences
- Common use cases
- Interview-ready explanations

For deeper learning, see the [Detailed TCP vs UDP Interview Guide](TCPvsUDP-DetailedInterviewGuide.md).

---

## 1. Where TCP and UDP Fit

TCP and UDP operate at the **Transport Layer**.

```text
Application Layer
HTTP, HTTPS, DNS, SMTP, WebSocket
        |
        v
Transport Layer
TCP, UDP
        |
        v
Network Layer
IP
        |
        v
Data Link Layer
Ethernet, Wi-Fi
```

- The **application layer** defines how applications exchange information.
- The **transport layer** delivers data between processes using port numbers.
- The **network layer** routes packets between devices using IP addresses.

Example:

```text
Browser
   |
   | HTTP
   v
TCP
   |
   | IP
   v
Web Server
```

HTTP defines the request and response format. TCP delivers the bytes reliably. IP routes the packets.

---

## 2. Transport-Layer Basics

### IP Address

An IP address identifies a device or network interface.

```text
192.168.1.10
```

### Port

A port identifies a process or application on that device.

```text
192.168.1.10:8080
```

Common ports:

| Protocol | Default Port |
|---|---:|
| HTTP | 80 |
| HTTPS | 443 |
| SSH | 22 |
| DNS | 53 |
| PostgreSQL | 5432 |
| MySQL | 3306 |

### Socket

A socket is a communication endpoint.

A network flow is commonly identified by:

```text
Source IP
Source Port
Destination IP
Destination Port
Transport Protocol
```

---

# 3. TCP

## What Is TCP?

TCP stands for **Transmission Control Protocol**.

TCP is:

- Connection-oriented
- Reliable
- Ordered
- Full-duplex
- Byte-stream based

TCP establishes a connection before application data is exchanged.

---

## TCP Three-Way Handshake

```text
Client                              Server
   |                                   |
   | -------- SYN -------------------> |
   | <------ SYN-ACK ----------------- |
   | -------- ACK -------------------> |
   |                                   |
   |       Connection Established      |
```

### SYN

The client requests a connection and sends an initial sequence number.

### SYN-ACK

The server acknowledges the client and sends its own initial sequence number.

### ACK

The client acknowledges the server. The connection is established.

The handshake confirms that both endpoints can send and receive data.

---

## How TCP Provides Reliability

TCP uses:

- Sequence numbers
- Acknowledgements
- Retransmission
- Checksums
- Duplicate detection
- Ordered delivery

Example:

```text
Sent:     1, 2, 3
Received: 1, 3, 2
Delivered to application: 1, 2, 3
```

If segment 2 is lost, TCP retransmits it.

---

## TCP Flow Control

Flow control prevents a fast sender from overwhelming a slow receiver.

The receiver advertises a **receive window**, which tells the sender how much data it can accept.

---

## TCP Congestion Control

Congestion control prevents TCP from overwhelming the network.

TCP adjusts its sending rate based on:

- Packet loss
- Round-trip time
- Duplicate acknowledgements
- Retransmission timeouts

Common concepts include:

- Slow start
- Congestion avoidance
- Fast retransmit
- Fast recovery

---

## TCP Head-of-Line Blocking

TCP delivers bytes in order.

If segment 3 is missing but segments 4 and 5 arrive, TCP waits for segment 3 before delivering later data to the application.

```text
Received: 1, 2, _, 4, 5
                 ^
              waiting for 3
```

This protects correctness but can increase latency.

---

## TCP Is a Byte Stream

TCP does not preserve application message boundaries.

If the application sends:

```text
Message A
Message B
```

TCP may deliver:

```text
Mess
age AMess
age B
```

The application protocol must define message framing using lengths, delimiters, or another rule.

---

## TCP Connection Termination

TCP commonly closes a connection using FIN and ACK messages.

```text
Client                              Server
   | -------- FIN -------------------> |
   | <------- ACK -------------------- |
   | <------- FIN -------------------- |
   | -------- ACK -------------------> |
```

Each communication direction is closed independently.

---

## Common TCP Use Cases

TCP is commonly used for:

- HTTP/1.1
- HTTP/2
- HTTPS
- REST APIs
- WebSockets
- Database connections
- SSH
- Email
- File transfers

Use TCP when complete and ordered delivery is required.

---

# 4. UDP

## What Is UDP?

UDP stands for **User Datagram Protocol**.

UDP is:

- Connectionless
- Message-oriented
- Lightweight
- Low-overhead
- Not reliable by itself
- Not ordered
- Without built-in retransmission

UDP sends independent units called **datagrams**.

---

## No Handshake

UDP can send data immediately.

```text
Sender
   |
   | Datagram
   v
Receiver
```

There is no three-way handshake.

---

## UDP Delivery Behavior

Suppose the sender transmits:

```text
1, 2, 3
```

Possible receiver outcomes include:

```text
1, 2, 3
```

```text
1, 3
```

```text
3, 1, 2
```

```text
1, 2, 2, 3
```

UDP does not guarantee:

- Delivery
- Ordering
- Duplicate removal
- Retransmission

The application must implement these features when required.

---

## UDP Preserves Message Boundaries

UDP is message-oriented.

If the sender sends two datagrams:

```text
Datagram A
Datagram B
```

The receiver processes them as two separate datagrams.

This differs from TCP's continuous byte stream.

---

## Why Use UDP?

UDP is useful when:

- Low latency is important
- Occasional loss is acceptable
- Old data becomes irrelevant quickly
- The application manages reliability
- Broadcast or multicast is required

---

## Real-Time Example

In a voice call, an audio packet that arrives late may no longer be useful.

```text
Packet 1 -> played
Packet 2 -> lost
Packet 3 -> played
Packet 4 -> played
```

A small audio gap may be better than delaying the entire conversation.

---

## Application-Level Reliability

An application or protocol can add selected reliability features over UDP:

- Sequence numbers
- Acknowledgements
- Retransmission
- Congestion control
- Forward error correction
- Encryption

This allows the application to decide which data must be reliable.

---

## QUIC and HTTP/3

QUIC is a modern transport protocol built over UDP.

```text
HTTP/3
   |
   v
QUIC
   |
   v
UDP
   |
   v
IP
```

QUIC adds:

- Reliable delivery
- Encryption
- Congestion control
- Multiple independent streams
- Faster connection establishment

UDP is the foundation, but QUIC supplies capabilities that raw UDP does not.

---

## Common UDP Use Cases

UDP is commonly used for:

- DNS
- Voice over IP
- Video calls
- WebRTC media
- Online gaming
- DHCP
- Telemetry
- Broadcast and multicast
- QUIC and HTTP/3

Use UDP when latency and freshness matter more than perfect transport-layer delivery.

---

# 5. TCP vs UDP

| Feature | TCP | UDP |
|---|---|---|
| Full name | Transmission Control Protocol | User Datagram Protocol |
| Connection | Connection-oriented | Connectionless |
| Reliability | Built in | Not built in |
| Ordering | Guaranteed | Not guaranteed |
| Retransmission | Yes | No |
| Duplicate handling | Yes | Application responsibility |
| Flow control | Yes | No |
| Congestion control | Yes | Not provided by raw UDP |
| Data model | Byte stream | Datagrams |
| Message boundaries | Not preserved | Preserved |
| Handshake | Three-way handshake | None |
| Minimum header | 20 bytes | 8 bytes |
| Latency | Usually higher | Usually lower |
| Broadcast/multicast | No native support | Supported |
| Typical use | APIs, files, databases | Voice, video, games, DNS |

---

# 6. When to Choose TCP

Choose TCP when:

- Every byte must arrive
- Data must remain ordered
- Duplicate data must be removed
- File integrity matters
- Requests and responses must be complete
- You prefer transport-level reliability

Examples:

- Payment API
- Database query
- File download
- Email
- SSH session
- REST API call

---

# 7. When to Choose UDP

Choose UDP when:

- Low latency is critical
- Some packet loss is acceptable
- Old data quickly becomes useless
- The application can handle ordering or recovery
- Broadcast or multicast is needed

Examples:

- Voice call
- Video call
- Live game position updates
- DNS lookup
- Real-time telemetry
- WebRTC media

---

# 8. Applications Can Use Both

An online game might use:

```text
UDP:
- Player position
- Movement
- Aim direction

TCP or reliable messaging:
- Login
- Purchases
- Match results
- Inventory changes
```

The protocol can be selected separately for each type of data.

---

# 9. Common Scenarios

| Scenario | Typical Choice | Reason |
|---|---|---|
| REST API | TCP | Complete and ordered request/response |
| Payment API | TCP | Duplicate or missing data is unacceptable |
| File download | TCP | Every byte must arrive correctly |
| Database connection | TCP | Queries and results must be reliable |
| Chat messages | TCP/WebSocket | Messages should not disappear |
| Voice call | UDP | Low latency is more important than retransmitting old audio |
| Video call | UDP | Late frames are often useless |
| Online gaming | Often UDP | Fresh position data matters more than old data |
| DNS | Usually UDP | Small, fast queries; client can retry |
| HTTP/3 | UDP through QUIC | QUIC adds reliability and avoids some TCP limitations |

---

# 10. Common Interview Questions

- What is the difference between TCP and UDP?
- What does connection-oriented mean?
- Explain the TCP three-way handshake.
- Why does TCP require three handshake steps?
- How does TCP guarantee reliability?
- What are sequence and acknowledgement numbers?
- What is retransmission?
- What is flow control?
- What is congestion control?
- What is head-of-line blocking?
- Does TCP preserve message boundaries?
- Why is UDP faster in many scenarios?
- Does UDP guarantee delivery?
- Why is UDP used for voice and video?
- Why is TCP used for file transfer?
- Why does DNS usually use UDP?
- Can UDP be reliable?
- How does QUIC use UDP?
- Can one application use both TCP and UDP?
- Would you use TCP or UDP for a payment system?

---

# 11. Interview Readiness Checklist

## Fundamentals

- [ ] I can explain the transport layer.
- [ ] I understand IP addresses and ports.
- [ ] I can explain what a socket is.
- [ ] I understand how TCP and UDP relate to IP.

## TCP

- [ ] I can explain TCP in my own words.
- [ ] I can explain the three-way handshake.
- [ ] I understand sequence numbers and acknowledgements.
- [ ] I can explain retransmission.
- [ ] I understand ordered delivery.
- [ ] I can explain flow control.
- [ ] I can explain congestion control.
- [ ] I understand head-of-line blocking.
- [ ] I know that TCP is a byte stream.
- [ ] I can explain connection termination.

## UDP

- [ ] I can explain UDP in my own words.
- [ ] I understand connectionless communication.
- [ ] I know that UDP preserves datagram boundaries.
- [ ] I understand UDP's delivery limitations.
- [ ] I can explain why UDP is used for real-time applications.
- [ ] I can explain application-level reliability.
- [ ] I understand how QUIC uses UDP.

## Comparison

- [ ] I can compare TCP and UDP clearly.
- [ ] I can choose the correct protocol for common scenarios.
- [ ] I can explain reliability versus latency trade-offs.
- [ ] I can explain why an application may use both.

---

# Senior Interview Summary

> TCP and UDP are transport-layer protocols that provide process-to-process communication using port numbers. TCP is connection-oriented and provides reliable, ordered byte-stream delivery using sequence numbers, acknowledgements, retransmission, flow control, and congestion control. It is the preferred choice for APIs, database communication, file transfers, and other operations where complete and ordered delivery is required. UDP is connectionless and sends independent datagrams without built-in delivery, ordering, or retransmission guarantees. Its lower overhead makes it suitable for latency-sensitive workloads such as voice, video, online gaming, DNS, and telemetry. I select between them based on whether the requirement prioritizes reliability and ordering or latency and data freshness.
