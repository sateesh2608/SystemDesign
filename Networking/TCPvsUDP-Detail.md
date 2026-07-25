# TCP vs UDP — Detailed Interview Guide

This guide explains TCP and UDP in depth for backend engineering, networking fundamentals, system design, and senior software engineering interviews.

For quick revision, see [TCP vs UDP](TCPvsUDP.md).

---

# Table of Contents

1. [Networking Context](#1-networking-context)
2. [Transport-Layer Responsibilities](#2-transport-layer-responsibilities)
3. [Ports and Sockets](#3-ports-and-sockets)
4. [TCP Fundamentals](#4-tcp-fundamentals)
5. [TCP Three-Way Handshake](#5-tcp-three-way-handshake)
6. [TCP Reliability and Ordering](#6-tcp-reliability-and-ordering)
7. [TCP Flow Control](#7-tcp-flow-control)
8. [TCP Congestion Control](#8-tcp-congestion-control)
9. [TCP Connection Termination](#9-tcp-connection-termination)
10. [UDP Fundamentals](#10-udp-fundamentals)
11. [Reliability over UDP](#11-reliability-over-udp)
12. [Broadcast and Multicast](#12-broadcast-and-multicast)
13. [QUIC and HTTP/3](#13-quic-and-http3)
14. [TCP vs UDP Comparison](#14-tcp-vs-udp-comparison)
15. [Scenario-Based Selection](#15-scenario-based-selection)
16. [Production Considerations](#16-production-considerations)
17. [Interview Questions](#17-interview-questions)
18. [Interview Readiness Tracker](#18-interview-readiness-tracker)

---

# 1. Networking Context

TCP and UDP operate at the transport layer.

```text
Application Layer
HTTP, HTTPS, DNS, SMTP, WebSocket, gRPC
        |
        v
Transport Layer
TCP, UDP, QUIC
        |
        v
Internet Layer
IP, ICMP
        |
        v
Link Layer
Ethernet, Wi-Fi
```

The application layer defines the meaning of messages.

The transport layer moves data between processes.

The IP layer routes packets between devices and networks.

Example:

```text
REST API Request
      |
      v
HTTP
      |
      v
TCP
      |
      v
IP
      |
      v
Ethernet or Wi-Fi
```

---

# 2. Transport-Layer Responsibilities

The transport layer can provide:

- Process-to-process communication
- Port addressing
- Segmentation and reassembly
- Multiplexing and demultiplexing
- Reliability
- Ordering
- Flow control
- Congestion control
- Error detection

TCP provides most of these capabilities directly.

UDP provides a smaller and simpler transport service, leaving more responsibility to the application.

---

# 3. Ports and Sockets

## 3.1 Ports

An IP address identifies a device or interface.

A port identifies an application process.

```text
10.0.0.15:8080
```

Port ranges:

| Range | Purpose |
|---|---|
| 0–1023 | Well-known ports |
| 1024–49151 | Registered ports |
| 49152–65535 | Dynamic or ephemeral ports |

A client typically receives an ephemeral source port.

```text
Client: 192.168.1.20:53142
Server: 10.0.0.15:443
```

## 3.2 Socket

A socket is a software endpoint for network communication.

A connection or flow is commonly identified by:

```text
Source IP
Source Port
Destination IP
Destination Port
Protocol
```

This is often called a five-tuple.

## 3.3 Multiplexing and Demultiplexing

Many applications can use the same network interface simultaneously.

```text
Browser        -> Port 53142
Database Tool  -> Port 53143
Email Client   -> Port 53144
```

The operating system uses ports to route incoming data to the correct socket.

---

# 4. TCP Fundamentals

TCP stands for **Transmission Control Protocol**.

TCP is:

- Connection-oriented
- Reliable
- Ordered
- Full-duplex
- Byte-stream based
- Stateful

A TCP endpoint maintains:

- Connection state
- Sequence numbers
- Acknowledgement numbers
- Send and receive buffers
- Receive window
- Congestion window
- Retransmission timers

---

## 4.1 Important TCP Header Fields

A TCP segment includes fields such as:

- Source port
- Destination port
- Sequence number
- Acknowledgement number
- Flags
- Window size
- Checksum
- Options

Common flags:

| Flag | Meaning |
|---|---|
| SYN | Synchronize sequence numbers |
| ACK | Acknowledgement field is valid |
| FIN | Sender has finished sending |
| RST | Reset the connection |
| PSH | Request prompt delivery |
| URG | Urgent pointer is valid |

---

## 4.2 TCP Is a Byte Stream

TCP presents a continuous stream of bytes.

Application writes:

```text
write("HELLO")
write("WORLD")
```

The receiver may read:

```text
HELLOWORLD
```

or:

```text
HEL
LOWOR
LD
```

TCP preserves byte order but not message boundaries.

Applications must define framing through:

- Fixed-size messages
- Delimiters
- Length prefixes
- Protocol headers

---

# 5. TCP Three-Way Handshake

```text
Client                              Server
   |                                   |
   | SYN, Seq = X -------------------> |
   | <------------- SYN-ACK, Seq = Y   |
   |                Ack = X + 1         |
   | ACK, Ack = Y + 1 ----------------> |
   |                                   |
   |       Connection Established      |
```

## 5.1 SYN

The client requests a connection and sends its initial sequence number.

## 5.2 SYN-ACK

The server acknowledges the client's sequence number and sends its own initial sequence number.

## 5.3 ACK

The client acknowledges the server's sequence number.

## 5.4 Why Three Steps?

The handshake verifies that:

- The client can send
- The client can receive
- The server can send
- The server can receive
- Both sides agree on starting sequence numbers

## 5.5 SYN Flood

A SYN flood sends many incomplete connection requests to consume server resources.

Common protections include:

- SYN cookies
- Connection backlog tuning
- Rate limiting
- Firewalls
- Load balancers
- DDoS protection

---

# 6. TCP Reliability and Ordering

## 6.1 Sequence Numbers

TCP sequence numbers represent byte positions.

```text
Segment A: Sequence 1000, Length 500
Segment B: Sequence 1500, Length 500
Segment C: Sequence 2000, Length 500
```

## 6.2 Acknowledgements

TCP normally uses cumulative acknowledgements.

```text
ACK 2000
```

means that all bytes before 2000 were received.

## 6.3 Packet Loss and Retransmission

TCP detects loss through:

- Retransmission timeouts
- Duplicate acknowledgements
- Selective acknowledgements

If a segment is lost, TCP retransmits it.

## 6.4 Duplicate Acknowledgements

If byte 1500 is missing but later data arrives, the receiver may repeatedly send:

```text
ACK 1500
ACK 1500
ACK 1500
```

This can trigger fast retransmit.

## 6.5 Selective Acknowledgement

Selective Acknowledgement, or SACK, allows the receiver to identify later ranges that arrived successfully.

This helps the sender retransmit only missing ranges.

## 6.6 Duplicate Detection

If both an original and retransmitted segment arrive, TCP uses sequence numbers to discard duplicate bytes.

## 6.7 Ordered Delivery

TCP delivers bytes to the application in order.

```text
Arrival order:     1, 3, 2
Application order: 1, 2, 3
```

## 6.8 Head-of-Line Blocking

Suppose the receiver has:

```text
1, 2, _, 4, 5
```

TCP cannot deliver 4 and 5 until 3 arrives.

This protects correctness but can delay unrelated application data sharing the same stream.

---

# 7. TCP Flow Control

Flow control protects the receiver from a sender that is too fast.

The receiver advertises a **receive window**.

```text
Receive Window = 64 KB
```

The sender limits the amount of unacknowledged data based on this value.

## 7.1 Sliding Window

TCP can have multiple segments in flight.

```text
Acknowledged | In Flight | Allowed to Send | Blocked
```

As acknowledgements arrive, the window moves forward.

## 7.2 Zero Window

If the receive buffer is full, the receiver may advertise a zero window.

The sender pauses normal transmission and checks later to see whether space is available.

## 7.3 Window Scaling

Window scaling allows TCP to use larger receive windows on high-bandwidth networks.

---

# 8. TCP Congestion Control

Congestion control protects the network.

TCP typically tracks a congestion window in addition to the receiver's window.

The sender is limited by the smaller of:

```text
Receive Window
Congestion Window
```

## 8.1 Slow Start

TCP starts cautiously because it does not yet know network capacity.

The congestion window grows as acknowledgements arrive.

## 8.2 Congestion Avoidance

After reaching a threshold, TCP increases the sending rate more conservatively.

## 8.3 Fast Retransmit

Several duplicate acknowledgements can indicate that a segment was lost.

TCP retransmits before waiting for a full timeout.

## 8.4 Fast Recovery

Some TCP algorithms reduce the sending rate after loss without returning to the initial rate.

## 8.5 Common Algorithms

Examples include:

- Reno
- New Reno
- CUBIC
- BBR

## 8.6 Flow Control vs Congestion Control

| Topic | Flow Control | Congestion Control |
|---|---|---|
| Protects | Receiver | Network |
| Signal | Receive window | Loss, delay, acknowledgements |
| Goal | Avoid receiver overflow | Avoid network overload |

---

# 9. TCP Connection Termination

## 9.1 Four-Way Close

```text
Client                              Server
   | FIN ---------------------------> |
   | <-------------------------- ACK |
   | <-------------------------- FIN |
   | ACK ---------------------------> |
```

Each direction is closed separately because TCP is full-duplex.

## 9.2 Half-Close

One endpoint can stop sending while continuing to receive.

## 9.3 RST

A reset terminates a connection immediately.

Possible causes:

- No service is listening
- Application failure
- Invalid connection state
- Forced connection close

## 9.4 TIME_WAIT

The active closer commonly enters `TIME_WAIT`.

Purposes:

- Allow the final ACK to be retransmitted
- Prevent delayed packets from entering a new connection
- Ensure the old connection fully expires

Many short-lived connections can create many `TIME_WAIT` sockets.

Connection reuse and pooling help reduce this problem.

---

# 10. UDP Fundamentals

UDP stands for **User Datagram Protocol**.

UDP is:

- Connectionless
- Datagram-oriented
- Low-overhead
- Without built-in reliability
- Without built-in ordering
- Without built-in retransmission
- Without built-in flow control
- Without built-in congestion control

---

## 10.1 UDP Header

A UDP header contains:

- Source port
- Destination port
- Length
- Checksum

Its header is 8 bytes.

## 10.2 Datagram Boundaries

UDP preserves message boundaries.

Each send creates one datagram.

## 10.3 Possible Outcomes

A UDP datagram may be:

- Delivered
- Lost
- Duplicated
- Reordered
- Corrupted and discarded

## 10.4 UDP Checksum

The checksum detects accidental corruption.

It does not provide:

- Encryption
- Authentication
- Retransmission

## 10.5 No Handshake

UDP sends immediately without establishing a connection first.

## 10.6 No Built-In Backpressure

A sender can produce datagrams faster than the receiver or network can handle.

Buffers may overflow and datagrams may be dropped.

Applications must control their sending rate.

---

# 11. Reliability over UDP

Applications can add only the reliability features they need.

Possible mechanisms include:

- Sequence numbers
- Message IDs
- Acknowledgements
- Retransmission
- Deduplication
- Reordering buffers
- Forward error correction
- Rate control
- Congestion control

## 11.1 Selective Reliability

An online game might treat data differently.

```text
Player position:
- Unreliable
- Latest update wins

Purchase result:
- Reliable
- Must be acknowledged
- Must be deduplicated
```

## 11.2 Forward Error Correction

Forward error correction sends extra recovery information.

The receiver may reconstruct missing data without waiting for retransmission.

## 11.3 Jitter Buffer

Voice and video packets may arrive at uneven intervals.

A jitter buffer stores a small amount of data to smooth playback.

A larger buffer reduces interruptions but increases latency.

---

# 12. Broadcast and Multicast

## Unicast

```text
Sender -> One receiver
```

## Broadcast

```text
Sender -> All devices in a local broadcast domain
```

## Multicast

```text
Sender -> A subscribed group
```

UDP supports these communication styles.

TCP is designed primarily for one-to-one connections.

---

# 13. QUIC and HTTP/3

QUIC runs over UDP but provides advanced transport behavior.

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

QUIC includes:

- Reliable delivery
- Congestion control
- Encryption
- Multiple independent streams
- Faster connection setup
- Connection migration

## 13.1 Why UDP?

UDP is widely supported by operating systems and network infrastructure.

QUIC can evolve in user space instead of requiring kernel-level TCP changes.

## 13.2 Independent Streams

QUIC supports multiple logical streams.

Loss in one stream does not necessarily block unrelated streams.

## 13.3 Integrated Encryption

QUIC integrates modern TLS security into its connection establishment.

## 13.4 Connection Migration

QUIC can use connection identifiers instead of depending only on IP and port.

This helps a mobile device continue a connection when switching between Wi-Fi and cellular networks.

---

# 14. TCP vs UDP Comparison

| Feature | TCP | UDP |
|---|---|---|
| Connection model | Connection-oriented | Connectionless |
| State | Maintains connection state | Minimal transport state |
| Reliability | Built in | Application responsibility |
| Ordering | Built in | Application responsibility |
| Retransmission | Built in | Not built in |
| Flow control | Built in | Not built in |
| Congestion control | Built in | Not built in |
| Data model | Byte stream | Datagrams |
| Message boundaries | Not preserved | Preserved |
| Setup | Three-way handshake | No handshake |
| Minimum header | 20 bytes | 8 bytes |
| Broadcast/multicast | No native support | Supported |
| Typical use | APIs and file transfer | Real-time media and telemetry |

---

# 15. Scenario-Based Selection

## Payment API

Choose TCP.

However, TCP reliability does not prevent duplicate business requests. Use application-level idempotency keys.

## File Download

Choose TCP because every byte must arrive correctly and in order.

## Database Connection

Choose TCP because queries and results require reliable ordered delivery.

## Voice Call

Choose UDP-based real-time transport because late audio is often less useful than missing audio.

## Video Call

Choose UDP when possible. Real-time media may use jitter buffers, selective retransmission, and forward error correction.

## Online Game

Use both when appropriate.

UDP:

- Position
- Movement
- Aim direction

Reliable communication:

- Authentication
- Purchases
- Inventory
- Match completion

## DNS

DNS commonly uses UDP for ordinary requests.

TCP may be used for:

- Zone transfers
- Large responses
- Truncated UDP responses
- DNS over TLS

## Chat

Traditional WebSockets use TCP.

The application still needs:

- Message IDs
- Persistence
- Reconnection
- Acknowledgements
- Deduplication

## HTTP/3

HTTP/3 uses QUIC over UDP.

---

# 16. Production Considerations

## 16.1 Latency

Total latency may include:

- DNS lookup
- Connection establishment
- TLS handshake
- Network travel
- Server processing
- Retransmission

Connection reuse reduces repeated setup costs.

## 16.2 Packet Loss

TCP retransmits and may reduce its sending rate.

UDP applications decide whether to ignore, reconstruct, or retransmit lost data.

## 16.3 MTU and Fragmentation

Networks have a maximum transmission unit.

Oversized UDP datagrams are risky because losing one fragment can make the entire datagram unusable.

## 16.4 Keep-Alive

TCP keep-alive and application heartbeats help detect dead peers on long-lived connections.

## 16.5 Connection Pooling

Connection pools are commonly used for:

- Databases
- HTTP clients
- Message brokers

Benefits:

- Less handshake overhead
- Less TLS setup
- Fewer `TIME_WAIT` sockets
- Better throughput

## 16.6 NAT and Firewalls

NAT devices and firewalls maintain connection mappings and timeouts.

UDP mappings may expire quickly when idle.

Real-time systems often send heartbeats.

## 16.7 Observability

Useful TCP metrics:

- Active connections
- Connection failures
- Retransmissions
- Round-trip time
- Reset count
- Send and receive queue size
- `TIME_WAIT` count

Useful UDP metrics:

- Datagrams sent and received
- Packet loss
- Out-of-order rate
- Duplicate rate
- Jitter
- Buffer drops
- Application acknowledgements

## 16.8 Security

TCP and UDP do not automatically provide complete application security.

Examples:

```text
HTTPS -> TLS -> TCP
```

```text
HTTP/3 -> QUIC encryption -> UDP
```

```text
WebRTC media -> DTLS-SRTP -> UDP/TCP
```

Applications still need:

- Authentication
- Authorization
- Encryption
- Validation
- Rate limiting
- DDoS protection

---

# 17. Interview Questions

## Fundamentals

- What is the transport layer?
- What is a port?
- What is a socket?
- What is a five-tuple?
- What is multiplexing?
- How do TCP and UDP relate to IP?

## TCP

- Explain the TCP three-way handshake.
- Why is the handshake three steps?
- What are SYN, ACK, FIN, and RST?
- What are sequence numbers?
- What are cumulative acknowledgements?
- How does TCP detect packet loss?
- What is fast retransmit?
- What is SACK?
- What is flow control?
- What is a receive window?
- What is window scaling?
- What is congestion control?
- What is slow start?
- What is head-of-line blocking?
- Does TCP preserve message boundaries?
- Why does an application need framing over TCP?
- Explain the four-way close.
- What is a half-close?
- What is `TIME_WAIT`?
- How does connection pooling help?

## UDP

- What is a UDP datagram?
- Why is UDP connectionless?
- Does UDP guarantee delivery?
- Does UDP preserve message boundaries?
- Can UDP packets arrive out of order?
- How can an application detect missing packets?
- How can reliability be added over UDP?
- What is forward error correction?
- What is a jitter buffer?
- Why is UDP used for real-time communication?
- What are broadcast and multicast?

## QUIC

- What is QUIC?
- Why does QUIC use UDP?
- How does QUIC provide reliability?
- How does QUIC reduce stream-level head-of-line blocking?
- Why does HTTP/3 use QUIC?
- What is connection migration?

## Scenarios

- TCP or UDP for a payment system?
- TCP or UDP for a video call?
- Why do games often use both?
- Why does a database use TCP?
- Why does DNS usually use UDP?
- Can a reliable protocol be built over UDP?
- Does TCP eliminate duplicate API requests?
- What happens when a TCP segment is lost?
- What happens when a UDP audio packet is lost?
- What metrics would you monitor?

---

# 18. Interview Readiness Tracker

## Fundamentals

- [ ] I understand the transport layer.
- [ ] I can explain ports and ephemeral ports.
- [ ] I can explain sockets and the five-tuple.
- [ ] I understand multiplexing and demultiplexing.

## TCP

- [ ] I can explain the three-way handshake.
- [ ] I understand SYN, ACK, FIN, and RST.
- [ ] I understand sequence and acknowledgement numbers.
- [ ] I can explain retransmission and SACK.
- [ ] I understand ordered delivery.
- [ ] I can explain head-of-line blocking.
- [ ] I understand flow control and receive windows.
- [ ] I understand congestion control and slow start.
- [ ] I know TCP does not preserve message boundaries.
- [ ] I can explain the four-way close.
- [ ] I understand `TIME_WAIT`.
- [ ] I can explain connection pooling.

## UDP

- [ ] I can explain UDP datagrams.
- [ ] I understand loss, duplication, and reordering.
- [ ] I know UDP preserves message boundaries.
- [ ] I can explain application-level reliability.
- [ ] I understand jitter buffers.
- [ ] I understand forward error correction.
- [ ] I can explain broadcast and multicast.
- [ ] I know why oversized datagrams are risky.

## QUIC

- [ ] I can explain QUIC at a high level.
- [ ] I understand why QUIC uses UDP.
- [ ] I know how QUIC adds reliability and encryption.
- [ ] I understand independent streams.
- [ ] I can explain HTTP/3 at a high level.
- [ ] I understand connection migration.

## Selection

- [ ] I can choose TCP or UDP for common scenarios.
- [ ] I can explain reliability versus latency trade-offs.
- [ ] I can explain why one application may use both.
- [ ] I understand that TCP does not replace application idempotency.
- [ ] I can discuss production metrics for both protocols.

---

# Senior Interview Answer

> TCP and UDP are transport-layer protocols that use port numbers to deliver data between applications. TCP is connection-oriented and stateful. It provides a reliable, ordered byte stream through sequence numbers, acknowledgements, retransmission, flow control, and congestion control. This makes it suitable for APIs, database connections, file transfers, and other workloads where complete and ordered delivery is required. UDP is connectionless and datagram-oriented. It does not provide built-in delivery, ordering, retransmission, flow control, or congestion control, which reduces overhead and gives the application more control. It is therefore common in voice, video, gaming, DNS, and telemetry, where low latency and freshness may be more important than recovering every packet. Modern protocols such as QUIC show that reliability, encryption, and congestion control can also be built over UDP.
