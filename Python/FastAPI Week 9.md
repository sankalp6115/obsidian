# FastAPI Week 9: WebSockets & Real-Time Features - Concept-Focused Guide

**Duration:** 13 hours **Learning Style:** Protocols and design trade-offs first, code lookups via documentation **Prerequisites:** Weeks 1-8 completed **Goal:** Understand how real-time communication works on the web, when each technique fits, and what makes live systems fail

---

## Table of Contents

1. [Why Request-Response Is Not Enough - 1.5 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#why-request-response-is-not-enough---15-hours)
2. [Real-Time Techniques Compared - 2 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#real-time-techniques-compared---2-hours)
3. [The WebSocket Protocol - 2.5 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#the-websocket-protocol---25-hours)
4. [Connection Lifecycle and Heartbeats - 2 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#connection-lifecycle-and-heartbeats---2-hours)
5. [Broadcasting and Pub/Sub - 2 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#broadcasting-and-pubsub---2-hours)
6. [Scaling Real-Time Systems - 1.5 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#scaling-real-time-systems---15-hours)
7. [Authentication, Reliability and Failure - 1.5 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#authentication-reliability-and-failure---15-hours)

---

# WHY REQUEST-RESPONSE IS NOT ENOUGH - 1.5 HOURS

## Part 1: The Model HTTP Was Built For

### Formal Definition: Request-Response

**Request-response:** A communication pattern where a client initiates every exchange and the server can only reply to a request it has received.

```
Client: "Give me post 42"
Server: "Here is post 42"
Client: (silence until the next question)

The server has no way to speak first.
```

This model fits most APIs: fetch a resource, submit a form, update a record.

### The Problem Real-Time Features Expose

```
Scenario: A user is reading a post. A new comment is added by someone else.

With request-response:
- The reader must ask "any new comments?" repeatedly.
- Most answers are "no", which wastes requests.
- Between two checks, the reader sees stale data.

Real-time need: the server delivers the new comment when it happens.
```

### Formal Definition: Real-Time Communication

**Real-time communication:** Delivering server-originated messages to clients with low latency, without the client having to ask for each one.

Two properties matter: **latency** (how quickly a message arrives) and **direction** (who may initiate a message).

---

# REAL-TIME TECHNIQUES COMPARED - 2 HOURS

## Part 1: Four Approaches

### Technique 1: Short Polling

```
Client asks every N seconds: "anything new?"

Pros: Works everywhere, trivially simple, stateless server.
Cons: Latency bounded by the interval. Many empty responses.
      Load scales with clients times frequency, not with actual events.

Use when: updates are rare, latency of several seconds is acceptable.
```

### Technique 2: Long Polling

```
Client asks: "anything new?"
Server holds the request open until something happens or a timeout occurs.
Client immediately asks again.

Pros: Near-instant delivery, works through most proxies, plain HTTP.
Cons: Each waiting client holds a connection open. Reconnect logic is required.
      Ordering and missed-message handling must be designed explicitly.

Use when: you need low latency but cannot use WebSockets or SSE.
```

### Technique 3: Server-Sent Events (SSE)

```
Client opens one HTTP connection. Server streams events down it.
Only server to client direction.

Pros: Plain HTTP, automatic reconnection in browsers, event IDs for resume,
      simple to reason about.
Cons: One direction only. Limited by per-browser connection caps on HTTP/1.1.

Use when: notifications, live feeds, progress updates, anything where
the client rarely needs to send messages back.
```

### Technique 4: WebSockets

```
Client and server open one persistent, full-duplex connection.
Either side may send messages at any time.

Pros: Lowest overhead per message, bidirectional, binary and text support.
Cons: Stateful connections complicate scaling. Proxies and load balancers
      need explicit support. Reconnection and ordering are your responsibility.

Use when: chat, collaborative editing, multiplayer state, high-frequency
bidirectional traffic.
```

## Part 2: Choosing Among Them

```
Question 1: Does the client need to send frequent messages back?
├─ Yes → WebSockets
└─ No  → Question 2

Question 2: Is latency under a second required?
├─ Yes → SSE (or long polling if SSE is not available)
└─ No  → Short polling is often enough

Question 3: Is the update rate high and per-user?
├─ Yes → Reconsider: WebSockets with careful batching
└─ No  → Stay simple

Principle: Choose the least powerful technique that meets the requirement.
Each step up adds operational cost.
```

### Concept: Simplicity Is a Feature

Many "real-time" features, such as a notification bell, need only SSE or occasional polling. Reaching for WebSockets by default creates stateful infrastructure that must be scaled, monitored and secured.

---

# THE WEBSOCKET PROTOCOL - 2.5 HOURS

## Part 1: Formal Definition

### Definition: WebSocket

**WebSocket (RFC 6455):** A protocol providing a persistent, full-duplex, message-oriented communication channel over a single TCP connection, established by an HTTP-compatible handshake.

Key ideas:

- **Starts as HTTP.** The connection begins as an ordinary HTTP request.
- **Upgrades.** Both sides agree to switch protocols.
- **Message-oriented.** Data travels in frames that form messages, rather than as a byte stream you must parse.
- **Full-duplex.** Both sides may send at any time, independently.

## Part 2: The Handshake

```
Client sends an HTTP GET with headers:
    Upgrade: websocket
    Connection: Upgrade
    Sec-WebSocket-Key: <random base64 value>
    Sec-WebSocket-Version: 13

Server responds:
    HTTP 101 Switching Protocols
    Upgrade: websocket
    Connection: Upgrade
    Sec-WebSocket-Accept: <derived from the client's key>

After 101, the TCP connection is no longer speaking HTTP.
It speaks WebSocket frames until either side closes it.
```

**Why the key exchange exists:** It proves the server understood the WebSocket handshake, and it prevents some non-WebSocket intermediaries from being confused into treating the stream as HTTP. It is not an authentication mechanism.

## Part 3: Frames and Messages

```
A message is composed of one or more frames.
Frames carry: opcode (text, binary, close, ping, pong), payload length, payload.

Control frames:
- Ping / Pong: liveness checks (see heartbeats below)
- Close: graceful shutdown, with an optional status code and reason

Data frames:
- Text: UTF-8 payload
- Binary: arbitrary bytes
```

**Concept:** Applications usually define their own message format on top, most often JSON with a `type` field. The protocol delivers messages; meaning is your design.

## Part 4: WebSocket and HTTP Compared

```
HTTP                              WebSocket
─────────────────────────         ─────────────────────────
Request → response, then done     Connection persists
Stateless by default              Stateful for the connection's life
Headers on every message          Small frame header per message
Client always initiates           Either side initiates
Caches and middleware apply       Many HTTP middlewares do not apply
```

**Important consequence:** Middleware you wrote for HTTP in Week 4 (request logging, per-request auth headers) does not automatically run per WebSocket message. Authentication and logging must be designed for the connection, and for each message where needed.

## Part 5: Origin Checking

```
Browsers send an Origin header on the WebSocket handshake.
Unlike CORS, the browser does NOT block the connection for you.
Any website can open a WebSocket to your server, and the browser attaches
the user's cookies unless SameSite prevents it.

Therefore: the server must check the Origin header itself,
against an allowlist, during the handshake.

This is "cross-site WebSocket hijacking" and it connects directly to the
CSRF discussion in Week 8.
```

---

# CONNECTION LIFECYCLE AND HEARTBEATS - 2 HOURS

## Part 1: Lifecycle Stages

```
1. Connecting:   Handshake in progress
2. Open:         Messages flow in both directions
3. Closing:      One side sent a close frame, awaiting acknowledgement
4. Closed:       TCP connection terminated

Each stage is a place where your code must have a defined behavior.
```

## Part 2: Dead Connections

### Concept: The Silent Failure

```
A phone walks into a tunnel. A laptop sleeps. A NAT mapping expires.
In each case, the TCP connection may look open on the server
for minutes, even though no packets can reach the client.

The server does not know the client is gone until it tries to send
and fails, or until a timeout expires.
```

### Concept: Heartbeats

**Heartbeat:** Periodic small messages (ping/pong or application-level) that verify a connection is still alive.

```
Server sends ping every 30 seconds.
If no pong returns within a timeout, the server closes the connection
and cleans up its state.

Also: intermediaries (load balancers, proxies) often close idle connections
after a fixed time. Heartbeats keep the connection from looking idle.
```

## Part 3: Reconnection on the Client

```
When a connection drops, the client should:
- Wait, then reconnect (exponential backoff with jitter)
- Avoid a thundering herd: many clients reconnecting at the same instant
- Re-subscribe to whatever it was watching
- Fetch missed state, because messages sent during the outage are lost
```

**Principle:** A WebSocket is a delivery channel, not a source of truth. After reconnection, the client should reconcile with the server's current state through a normal HTTP request. This is the single most common real-time bug: clients assume the socket never misses anything.

## Part 4: Backpressure

### Concept: Slow Consumers

```
Server sends 100 messages per second.
Client processes 10 per second.
Messages queue up in memory.

Eventually: memory exhaustion, or the process is killed.

Responses (design choices):
- Bound the per-connection queue; drop or disconnect when full
- Coalesce updates (send only the latest state, not every intermediate one)
- Throttle high-frequency sources
```

---

# BROADCASTING AND PUB/SUB - 2 HOURS

## Part 1: The Core Problem

```
A new comment is created on post 42.
Who needs to know? Every client currently viewing post 42.

Naive approach: loop over all connected sockets and send.
Problem: the loop only knows sockets in THIS process.
```

## Part 2: Rooms and Topics

### Concept: Topic-Based Routing

```
Clients subscribe to topics (channels, rooms):
  "post:42"        → viewers of post 42
  "user:7:notify"  → notifications for user 7

Publishers send to a topic. The system delivers to subscribers.
Publishers don't know who the subscribers are.
```

This is the **publish/subscribe** pattern: decoupling producers from consumers.

### Concept: Subscription Authorization

```
Subscribing is an authorization decision.
A client must not subscribe to "user:8:notify" just by naming it.

Week 8 applies here: function-level check (may this user read comments?)
and object-level check (may this user see post 42?) happen at subscribe time,
and ideally again periodically or on permission change.
```

## Part 3: Delivery Semantics

```
At-most-once:   Sent once, may be lost. Cheap. Fine for presence indicators.
At-least-once:  Retried until acknowledged, may duplicate. Clients must dedupe.
Exactly-once:   Very hard across networks. Usually approximated by
                at-least-once delivery plus idempotent handling.

Practical rule: Give every message an ID. Make client handling idempotent.
Use sequence numbers per topic if ordering matters, so gaps are detectable.
```

**Connection to Week 5:** Idempotent handling resembles designing database writes so that retries do not create duplicate rows.

---

# SCALING REAL-TIME SYSTEMS - 1.5 HOURS

## Part 1: The Single-Process Illusion

```
One FastAPI process, one in-memory dictionary of connections:
works fine for development and small deployments.

With several worker processes or several servers:
- User A is connected to server 1.
- User B is connected to server 2.
- A comment posted via server 2 must reach A on server 1.

In-memory state does not cross process boundaries.
```

## Part 2: The Shared Backplane

### Concept: Pub/Sub Backplane

```
A shared system (commonly Redis pub/sub or a message broker) sits between servers.

Flow:
1. Server 2 publishes "new comment on post 42" to the backplane.
2. Every server receives it.
3. Each server delivers it to its own local sockets subscribed to post 42.

Each server holds only its own connections.
The backplane carries the events.
```

### Concept: Sticky Sessions

```
A WebSocket must stay on the server where its handshake happened.
Load balancers need either:
- Sticky routing (same client to same server), or
- Stateless routing with the backplane handling cross-server delivery.

Read your load balancer's documentation for its WebSocket support.
```

## Part 3: Capacity Thinking

```
Each open connection costs memory and file descriptors.
Questions to answer before scaling:
- How many concurrent connections per process?
- How many messages per second, per connection, on average and at peak?
- What happens when a single topic has ten thousand subscribers?
  (fan-out cost grows with subscribers per event)

Measure with load tools before deciding on architecture.
```

---

# AUTHENTICATION, RELIABILITY AND FAILURE - 1.5 HOURS

## Part 1: Authenticating a Connection

### Concept: Authenticate Once, Authorize Continuously

```
The handshake is an HTTP request, so the token can be checked there.
Options for carrying the token:
- Query parameter: convenient, but tokens appear in server logs and history.
- First message after connecting: keeps the token out of URLs.
- Cookie: works for same-site browser clients; watch CSRF (Week 8).

Principle: prefer a token sent after the handshake, or a short-lived
ticket issued over HTTP, rather than a long-lived token in the URL.
```

### Concept: Token Expiry on a Live Connection

```
Problem: a JWT (Week 7) expires after 15 minutes, but the socket stays open.

Options:
- Close the connection when the token expires, forcing re-authentication.
- Require the client to send a fresh token periodically.
- Re-check permissions on each subscription or sensitive message.

A live connection is a long-lived authorization decision.
Plan for permission revocation to take effect promptly.
```

## Part 2: Abuse Controls

```
Real-time endpoints are easy to abuse:
- Flooding messages (rate limit per connection)
- Opening thousands of sockets (limit per user and per IP)
- Oversized messages (enforce maximum frame size)
- Subscribing to everything (limit subscriptions per connection)

These are the Week 8 concerns of resource exhaustion, applied to a connection.
```

## Part 3: Failure Modes to Design For

```
Failure                          Design response
───────────────────────────      ─────────────────────────────────────
Client disconnects silently      Heartbeats, server-side timeout
Client misses messages           Sequence numbers, reconcile over HTTP
Server crashes                   Clients reconnect and resync
Backplane is down                Degrade: fall back to polling, alert
Slow client                      Bounded queues, coalescing, disconnect
Duplicate delivery               Message IDs, idempotent handlers
```

### Concept: Graceful Shutdown

```
On deploy, servers restart. Every connected client drops at once.
If all clients reconnect simultaneously, the new server is flattened.

Mitigations:
- Send a "going away" close with a reason, so clients back off
- Stagger reconnect with jitter
- Drain connections gradually where the platform allows
```

---

# KEY CONCEPTS SUMMARY

## Real-Time Techniques

- Short polling: simplest, latency bounded by interval
- Long polling: low latency over plain HTTP, holds connections
- SSE: server-to-client streams, automatic reconnection in browsers
- WebSockets: full-duplex, stateful, most operational cost
- Choose the least powerful technique that meets the need

## WebSocket Protocol

- Starts as HTTP, upgraded via 101 Switching Protocols
- Message-oriented frames: text, binary, ping, pong, close
- Middleware for HTTP does not automatically apply per message
- Check the Origin header yourself

## Reliability

- Heartbeats detect silent death
- A socket is a delivery channel, not a source of truth
- Reconcile state over HTTP after reconnection
- Backpressure: bound queues, coalesce, disconnect slow consumers

## Broadcasting and Scaling

- Topics and pub/sub decouple publishers from subscribers
- Subscription is an authorization decision
- Delivery semantics: plan for at-least-once and idempotent handling
- Multiple servers need a shared backplane; in-memory state stays local

## Security

- Authenticate at handshake or first message; avoid long-lived tokens in URLs
- Live connections need expiry and revocation handling
- Rate-limit messages, connections and subscriptions

---

# WEEK 9 CHECKPOINT PROJECT

## Project: Live Comments and Notifications for the Blog API

### Objective

Add real-time behavior to the hardened Blog API from Week 8: new comments appear live for viewers of a post, and authors receive notifications.

### Part 1: Technique Decision Document

For each feature, choose and justify a technique:

- Live comments on a post viewed by many readers
- Notification bell for an author
- Collaborative editing of a draft (if you consider it)

Write the trade-offs: latency, direction, operational cost, fallback.

### Part 2: Topic and Authorization Design

- Define topic names (for example, `post:{id}` and `user:{id}:notify`)
- For each topic, state the subscribe-time check (Week 8 function-level and object-level)
- State what happens when a user loses access while subscribed

### Part 3: Message Contract

- Define a message envelope: type, ID, sequence, timestamp, payload
- Decide delivery semantics and how clients dedupe
- Write the reconnect procedure: what the client fetches over HTTP after reconnecting

### Part 4: Failure Analysis

Walk through these scenarios and write the expected behavior:

- A client's phone sleeps for ten minutes
- The server deploys and every socket drops
- Two server instances, one comment posted on one instance
- A malicious client subscribes to a topic it has no rights to
- A slow client falls behind by thousands of messages

### Part 5: Scaling Plan

- Explain how a second server instance receives events
- Identify where sticky routing is required
- Estimate connection memory and fan-out for a popular post

### Evaluation Checklist

**Conceptual understanding:**

- [ ] Can explain when SSE is better than WebSockets
- [ ] Can describe the 101 upgrade handshake
- [ ] Can explain why a socket is not a source of truth
- [ ] Can explain why per-process connection dictionaries fail at scale
- [ ] Can name the origin-checking requirement and why it exists

**Design:**

- [ ] Every subscription has an authorization check
- [ ] Clients can recover missed messages through HTTP
- [ ] Heartbeats and backpressure are addressed
- [ ] Tokens are not exposed in URLs or logs

---

# RESOURCES FOR DEEPER LEARNING

**Standards:**

- RFC 6455 (The WebSocket Protocol)
- WHATWG HTML Living Standard, section on Server-Sent Events
- RFC 9700 (OAuth 2.0 security best current practice, for token handling)

**Concepts:**

- "Designing Data-Intensive Applications" (stream processing and logs)
- Articles on the "thundering herd" problem and jittered backoff

**Tools for exploration:**

- Browser DevTools, Network tab, filter by WS, to inspect frames
- `websocat` or similar CLI tools to talk to a socket manually
- A load-testing tool that supports WebSockets, to measure fan-out

---

**End of Week 9 Content**

Real-time systems fail in quiet ways: a missed message, a dead connection nobody noticed, a slow client that quietly fills memory. Designing for those failures is most of the work.

**Next week:** Background tasks, streaming and advanced patterns, or do you want to revisit any Week 9 concept first?