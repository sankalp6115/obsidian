# FastAPI Week 10: Background Tasks, Streaming & Advanced Patterns - Concept-Focused Guide

**Duration:** 13 hours **Learning Style:** Architecture and failure models first, code lookups via documentation **Prerequisites:** Weeks 1-9 completed **Goal:** Know when work belongs outside the request, how to make that work reliable, how to stream large responses, and how async concurrency actually behaves

---

## Table of Contents

1. [Why Some Work Must Leave the Request - 1.5 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#why-some-work-must-leave-the-request---15-hours)
2. [In-Process Background Work vs Task Queues - 2.5 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#in-process-background-work-vs-task-queues---25-hours)
3. [Reliability: Retries, Idempotency, Dead Letters - 2.5 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#reliability-retries-idempotency-dead-letters---25-hours)
4. [Scheduling and Periodic Work - 1 Hour](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#scheduling-and-periodic-work---1-hour)
5. [Streaming Responses and Large Payloads - 2 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#streaming-responses-and-large-payloads---2-hours)
6. [Concurrency Models in Async Python - 2 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#concurrency-models-in-async-python---2-hours)
7. [Advanced API Patterns - 1.5 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#advanced-api-patterns---15-hours)

---

# WHY SOME WORK MUST LEAVE THE REQUEST - 1.5 HOURS

## Part 1: The Request Budget

### Formal Definition: Request Budget

**Request budget:** The time a client and its intermediaries are willing to wait for a response, typically a few seconds, after which timeouts, retries, or user abandonment occur.

```
Work inside a request:
- Validate input           (microseconds)
- Read or write the DB     (milliseconds)
- Call a payment API       (hundreds of milliseconds, may fail)
- Generate a PDF report    (seconds to minutes)
- Send an email            (seconds, may fail, may be slow)
- Resize an uploaded video (minutes)

The last three do not belong in the request budget.
```

### The Core Principle

**Separate acknowledgement from completion.** The request should confirm that work was accepted and durably recorded, then return. The work itself happens later, observably, and can be retried.

```
Synchronous:   POST /reports → generate PDF → return PDF   (client waits minutes)
Asynchronous:  POST /reports → record job, return 202 + job ID
               later:           worker generates PDF
               client:          GET /jobs/{id} → status, result link
```

The HTTP status **202 Accepted** exists precisely for this pattern: the request was accepted for processing, but processing is not finished.

## Part 2: What Qualifies as Background Work

```
Good candidates:
- Sending notifications, emails, webhooks
- Generating reports, exports, thumbnails
- Calling slow or unreliable third-party APIs
- Cleanup and maintenance (expired tokens, old drafts)

Poor candidates:
- Anything the client needs in the same response to proceed
- Operations that must be atomic with the request's database write
  (though an outbox pattern, below, can bridge this)
```

### Concept: The Outbox Pattern

```
Problem: You save a post to the database, then enqueue "send notifications".
If the enqueue fails after the commit, notifications are never sent.
If the enqueue succeeds before the commit, you notify about a post that was never saved.

Outbox: Write the event into an "outbox" table in the SAME transaction as the post.
A separate process reads the outbox and publishes events to the queue.

Result: The event exists if and only if the data change committed.
Connects directly to ACID from Week 5.
```

---

# IN-PROCESS BACKGROUND WORK VS TASK QUEUES - 2.5 HOURS

## Part 1: In-Process Background Tasks

### Concept: Work Scheduled Inside the Web Process

```
FastAPI can run a function after the response is sent,
within the same server process.

Properties:
- Zero infrastructure: no broker, no worker process
- Tied to the lifetime of the web process
- Lost if the process crashes or restarts mid-task
- Not retried automatically
- Competes with request handling for CPU and memory
```

### When In-Process Is Appropriate

```
Use when ALL of these hold:
- Losing the task occasionally is acceptable
- The task is short (seconds, not minutes)
- Volume is low
- You want simplicity over durability

Examples: logging a metric, invalidating a small cache entry,
sending a low-stakes analytics ping.
```

### When In-Process Is Not Appropriate

```
- Payments, legal notices, or anything that must eventually happen
- Long-running jobs (they block shutdown and slow other requests)
- CPU-heavy work (blocks the event loop or starves threads)
- Anything that must survive a deploy
```

## Part 2: Task Queues

### Formal Definition: Task Queue

**Task queue:** A system in which producers enqueue descriptions of work, and separate worker processes consume and execute them, with the queue holding work durably until it is completed.

### The Three Components

```
Producer (your API):    Enqueues a message describing the job
Broker (queue storage): Holds messages durably (Redis, RabbitMQ, SQS, etc.)
Worker (consumer):      Pulls messages, runs the job, acknowledges completion

Key property: the producer and worker are decoupled in time and in process.
The API can respond immediately; a worker can run on another machine.
```

### What Travels in a Message

```
Send identifiers and parameters, not large payloads or live objects.

Good:  {"job": "send_welcome_email", "user_id": 7}
Bad:   {"job": "...", "user": <entire ORM object>, "file": <50 MB bytes>}

Reasons:
- Live objects cannot be serialized reliably
- Large messages slow the broker
- The worker should load current state from the source of truth
  (a stale snapshot may already be wrong)
```

### Concept: Acknowledgement Timing

```
Auto-ack: broker considers the message done when delivered.
          If the worker dies mid-job, the job is lost.

Late-ack: broker considers the message done only after the worker finishes.
          If the worker dies, the message is redelivered.

Trade-off: late-ack gives at-least-once execution, which means jobs may run twice.
This is why idempotency (below) is essential.
```

## Part 3: Choosing a Model

```
Question 1: Must the work survive a process crash or deploy?
├─ No  → In-process background task
└─ Yes → Question 2

Question 2: Does it need retries, scheduling, or monitoring?
├─ Yes → Task queue with workers
└─ No  → Question 3

Question 3: Is it a single short job with a single consumer?
├─ Yes → Still consider a queue; durability is cheap insurance
└─ No  → Task queue

Principle: Start with in-process for truly disposable work.
Anything you would be upset to lose gets a queue.
```

---

# RELIABILITY: RETRIES, IDEMPOTENCY, DEAD LETTERS - 2.5 HOURS

## Part 1: Why Jobs Fail

```
Transient failures:  Network blip, rate limit, database deadlock, deploy in progress
Permanent failures:  Invalid input, deleted resource, logic bug

Retry transient failures. Do not retry permanent ones forever.
Classifying a failure correctly is the first design decision.
```

## Part 2: Retries

### Concept: Exponential Backoff with Jitter

```
Attempt 1 fails → wait ~1s
Attempt 2 fails → wait ~2s
Attempt 3 fails → wait ~4s
...
Add random jitter to each wait.

Why exponential: gives struggling dependencies room to recover.
Why jitter: prevents many failed jobs from retrying in synchronized waves
            (the thundering herd from Week 9, now on the server side).
```

### Concept: Retry Limits

```
Every retry policy needs:
- A maximum attempt count
- A maximum total duration
- A classification of which errors are retryable

Unbounded retries turn a single outage into a permanent backlog.
```

## Part 3: Idempotency

### Formal Definition: Idempotent Operation

**Idempotent operation:** An operation that produces the same final state whether it is performed once or many times.

```
Not idempotent: "Charge the card $10"
  Run twice → charged $20.

Idempotent:     "Set order 42 status to paid"
  Run twice → still paid.

Idempotent by design:  "Charge the card $10 for idempotency key K-123"
  Payment provider remembers K-123 and returns the original result on repeat.
```

### Why It Matters Here

```
Late-ack queues execute at-least-once.
Retries re-execute failed-looking jobs that actually succeeded.
Network timeouts make clients retry requests that the server already processed.

Therefore: every side-effecting job must either be naturally idempotent,
or record its own completion and skip work already done.
```

### Pattern: Idempotency Keys

```
Client sends a unique key with a request (Idempotency-Key header).
Server stores the key with the result.
A repeated request with the same key returns the stored result, without redoing work.

Scope: keys should expire after a reasonable window.
Connects to Week 5: this is the same idea as a unique constraint preventing duplicate rows.
```

## Part 4: Dead Letter Queues

### Formal Definition: Dead Letter Queue (DLQ)

**DLQ:** A separate queue where messages go after exhausting their retry policy, preserving them for inspection rather than discarding them.

```
Flow:
Job fails → retry → retry → ... → max attempts reached → moved to DLQ

Operators then:
- Inspect why it failed
- Fix the underlying cause
- Replay messages from the DLQ

Principle: a failed job is information. Do not silently delete it.
```

## Part 5: Observability for Jobs

```
Every job should be able to answer:
- Was it enqueued? When? By what request?
- How many attempts? What error on each?
- Did it complete? How long did it take?
- Which job and request caused it? (correlation IDs from Week 8 logging)

Metrics to watch: queue depth, age of oldest message, failure rate, retry rate.
A growing queue depth is an early warning long before users complain.
```

---

# SCHEDULING AND PERIODIC WORK - 1 HOUR

## Concept: Time-Driven vs Event-Driven

```
Event-driven:  "When a post is published, notify followers."
Time-driven:   "Every night at 02:00, delete drafts older than 90 days."

Time-driven work is commonly a scheduler that enqueues jobs at intervals,
not a loop inside the web process.
```

## Concept: Scheduling Pitfalls

```
Overlap:    A job takes longer than its interval. Two runs execute together.
            Solution: locks, or skip if the previous run is still active.

Multiple instances: Every web server runs the scheduler, so each job runs N times.
            Solution: run the scheduler as a single dedicated process,
            or use a distributed lock.

Missed runs: The scheduler was down at 02:00. Should it catch up?
            Decide per job: catch up, skip, or run once on recovery.

Time zones: Schedule in UTC internally; convert for display.
```

---

# STREAMING RESPONSES AND LARGE PAYLOADS - 2 HOURS

## Part 1: Why Stream

```
Without streaming:
Server builds the full response in memory → sends it all at once.
Memory use grows with the size of the data.
Client waits for the last byte before seeing the first.

With streaming:
Server produces chunks and sends each as ready.
Memory stays roughly constant.
Client can start processing immediately.
```

## Part 2: Streaming Mental Model

### Concept: Generators as Pipelines

```
A generator produces values one at a time, on demand.
A streaming response pulls from a generator and sends each chunk.

Pipeline:
Database cursor → row-by-row generator → serialize → chunks → network

Nothing needs to hold the full result set.
```

### Concept: Backpressure in Streams

```
If the client reads slowly, TCP flow control slows the sender.
A well-behaved stream therefore pauses production when the consumer is slow.

Warning: buffering in the middle (queues, lists) defeats this.
Any unbounded buffer between producer and network reintroduces the memory problem.
```

## Part 3: Common Streaming Use Cases

```
Large exports:        CSV or NDJSON of millions of rows
Log tailing:          Continuous lines for an admin tool
Model output:         Tokens arriving as generated
Progress reporting:   Server-Sent Events from Week 9, with event IDs
File downloads:       Chunked transfer of large binaries
```

### Concept: Framing Formats

```
Newline-delimited JSON (NDJSON): one JSON object per line.
  Easy to produce incrementally; easy for clients to parse line by line.

A single JSON array streamed piece by piece:
  Client cannot parse until the array closes, and an error mid-stream
  leaves invalid output.

Principle: choose a format whose every prefix is still meaningful.
```

## Part 4: Failure During a Stream

```
Once headers are sent, the status code is fixed (200 OK).
An error mid-stream cannot change it to 500.

Consequences:
- Clients must detect truncated streams (missing end marker, checksum, or error event)
- Design an explicit terminator or trailer so "complete" is distinguishable from "cut off"
```

## Part 5: Security Notes

```
- Streams can exhaust resources if a client holds a connection open slowly.
  Set timeouts and maximum stream durations.
- Exports are a data-exposure surface: apply the Week 8 object-level authorization
  per row, or filter the query before streaming.
```

---

# CONCURRENCY MODELS IN ASYNC PYTHON - 2 HOURS

## Part 1: The Event Loop

### Formal Definition: Event Loop

**Event loop:** A single-threaded scheduler that runs coroutines, switching between them whenever one awaits an I/O operation.

```
Coroutine A: sends DB query, awaits  → loop runs B
Coroutine B: sends HTTP call, awaits → loop runs C
Coroutine C: runs CPU work           → loop waits (nothing else runs)
DB result arrives → loop resumes A
```

### The Central Rule

```
Async helps with waiting (I/O-bound).
Async does NOT help with computing (CPU-bound).

A CPU-heavy function inside an async endpoint blocks the entire event loop:
every other request on that process waits.
```

This connects to Week 1: async/await does not make code parallel for computation.

## Part 2: Blocking Calls in Async Code

### Concept: The Silent Killer

```
async def endpoint():
    time.sleep(2)              # blocks the whole loop for 2 seconds
    data = requests.get(url)   # synchronous library, also blocks

Symptoms:
- Latency spikes across unrelated endpoints
- Throughput collapses under load
- Nothing errors; the system just becomes slow

Rule: inside async code, every I/O call must be awaitable,
or be moved off the loop.
```

### Concept: Moving Blocking Work Off the Loop

```
Option 1: Run blocking function in a thread pool.
          Good for blocking I/O libraries that lack async versions.

Option 2: Run CPU-bound work in a process pool.
          Escapes the GIL, at the cost of serialization overhead.

Option 3: Make the endpoint a plain def instead of async def.
          FastAPI runs plain def endpoints in a thread pool automatically.
          Simple, and often the right answer for blocking code.
```

### Decision: async def or def?

```
Endpoint uses only awaitable I/O (async DB driver, httpx async):  async def
Endpoint uses blocking libraries (sync ORM, requests):            def
Endpoint is CPU-heavy:                                            move to a worker process
Unsure:                                                           def is the safer default
```

## Part 3: The GIL

### Concept: Global Interpreter Lock

```
CPython allows one thread to execute Python bytecode at a time.

Consequences:
- Threads help with blocking I/O (the GIL is released while waiting).
- Threads do NOT speed up pure Python CPU work.
- Processes do, because each has its own interpreter and GIL.

Recent CPython versions are exploring removing the GIL;
check the current status in documentation before relying on it.
```

## Part 4: Structured Concurrency

### Concept: Tasks Need Owners

```
Fire-and-forget tasks:
  asyncio.create_task(work()) with no reference kept
  Problems: exceptions vanish, tasks may be garbage-collected mid-run,
            shutdown does not wait for them.

Structured concurrency: every task has a parent scope that waits for it,
handles its errors, and cancels it on shutdown.

Principle: every background coroutine should have an owner that
knows it exists and what happens if it fails.
```

### Concept: Cancellation and Timeouts

```
Cancellation is cooperative: a cancelled task receives an exception at its next await.
Code that catches broad exceptions can accidentally swallow cancellation.

Timeouts bound how long an operation may wait.
Every external call needs a timeout; the default is often "forever".
```

## Part 5: Lifespan and Shared Resources

### Concept: Startup and Shutdown

```
Resources that should exist for the whole process lifetime:
- Database connection pools
- HTTP client sessions
- Queue connections
- Loaded ML models or configuration

Create them once at startup, release them once at shutdown.
Creating them per request is a common and expensive mistake.
```

Connect to Week 4: this is the dependency system, but the resource's lifetime is the process, not the request.

---

# ADVANCED API PATTERNS - 1.5 HOURS

## Part 1: Pagination

### Offset Pagination

```
?page=3&size=20  →  OFFSET 40 LIMIT 20

Pros: Simple. Users can jump to page N.
Cons: Database scans and discards the offset rows: cost grows with depth.
      Inserts or deletes between pages cause skipped or repeated items.
```

### Cursor (Keyset) Pagination

```
?after=<opaque cursor>  →  WHERE (created_at, id) < (cursor values) ORDER BY ... LIMIT 20

Pros: Stable under concurrent writes. Consistent cost at any depth.
Cons: No random page access. Cursor must be opaque and sortable.

Principle: for feeds and logs, cursors. For small admin tables, offsets are fine.
```

Connect to Week 6: keyset pagination benefits from an index on the sort columns.

## Part 2: Versioning

### Concept: Versioning Is a Contract Decision

```
Breaking change: removing a field, changing a type, changing meaning.
Non-breaking change: adding an optional field, adding an endpoint.

Strategies:
- URL versioning (/v1/posts): explicit, easy to route, duplicates URLs
- Header or media-type versioning: cleaner URLs, harder to test in a browser
- Additive-only evolution: never break, deprecate slowly

Rule: prefer additive changes. Version only when a change truly breaks clients.
Deprecation needs a timeline communicated to clients, not just a code comment.
```

## Part 3: Long-Running Operations as Resources

```
Pattern: the job itself becomes a resource.

POST /exports          → 202 Accepted, Location: /exports/abc
GET  /exports/abc      → {"status": "running"}
GET  /exports/abc      → {"status": "done", "download": "/files/xyz"}

Clients poll the job resource, or subscribe to an event (Week 9).
The job resource is also where you record errors and retries.
```

## Part 4: Webhooks

### Concept: Outbound Events

```
Instead of clients polling you, you call them when something happens.

Design concerns:
- Delivery is at-least-once: receivers must be idempotent (event IDs)
- Sign payloads (HMAC) so receivers can verify origin
- Retry with backoff; move to DLQ after limits
- Receivers may be slow or down: never let webhook delivery block the request
- Provide a way for subscribers to replay missed events
```

This is the task-queue pattern from this week applied to outbound HTTP.

## Part 5: Rate Limiting Revisited

```
Week 8 introduced rate limiting for security.
Week 10 adds a different angle: fairness and capacity.

Patterns:
- Per-client quotas with clear response headers (remaining, reset time)
- Concurrency limits per tenant, so one heavy customer cannot starve others
- Shedding load: return 503 with Retry-After when overloaded, rather than queueing forever
```

---

# KEY CONCEPTS SUMMARY

## Background Work

- Separate acknowledgement (202 Accepted) from completion
- In-process tasks: disposable work only; they die with the process
- Task queues: producer, broker, worker; durable and retryable
- Outbox pattern links database commits to events reliably

## Reliability

- Classify failures as transient or permanent
- Exponential backoff with jitter, with bounded retries
- Late-ack gives at-least-once; idempotency makes it safe
- Idempotency keys prevent duplicate side effects
- Dead letter queues preserve failures for inspection

## Scheduling

- Time-driven work runs from a single scheduler, not every web instance
- Plan for overlap, missed runs, and time zones

## Streaming

- Generators and chunked responses keep memory bounded
- Backpressure must reach the producer; avoid unbounded buffers
- Headers are sent first, so mid-stream errors need explicit terminators

## Concurrency

- Async helps I/O waiting, not CPU work
- A blocking call in async code stalls the whole process
- Move blocking work to threads, processes, or plain def endpoints
- Every background task needs an owner, timeouts, and cancellation handling
- Create shared resources at startup, not per request

## API Patterns

- Cursor pagination for stability; offsets for simple cases
- Prefer additive API evolution; version only when breaking
- Model long-running work as a job resource
- Webhooks are at-least-once outbound events; receivers must be idempotent

---

# WEEK 10 CHECKPOINT PROJECT

## Project: Asynchronous Pipeline for the Blog API

### Objective

Extend the Week 9 Blog API so that publishing a post triggers reliable background work: notifications, a generated export, and an outbound webhook.

### Part 1: Work Classification

For each follow-up action of publishing a post, decide:

- In-process, queued, or scheduled?
- What happens if it fails? Retry, DLQ, or accept loss?
- Does it need the outbox pattern?

Write the decisions in a table with justifications.

### Part 2: Job Contract

- Define the job message: identifiers only, no large payloads
- Define idempotency: how a worker detects that a job already completed
- Define the retry policy: which errors are transient, the attempt limit, the backoff

### Part 3: Acknowledgement and Failure Analysis

Walk through these scenarios and write the expected behavior:

- Worker crashes after sending the email but before acknowledging
- The broker is unreachable when the post is saved
- The webhook receiver returns 500 for six hours
- A user publishes the same post twice in one second

### Part 4: Export Streaming Design

- Design an endpoint that streams all posts by one author as NDJSON
- Specify the terminator that distinguishes complete from truncated output
- Specify authorization applied per row
- Specify timeouts and maximum duration

### Part 5: Concurrency Review

- List every blocking call in your existing endpoints
- For each, decide: async driver, thread offload, or plain def
- Identify background coroutines and their owners

### Part 6: API Pattern Choices

- Choose pagination for the post feed and justify it
- Describe how the export job is modeled as a resource
- Describe your versioning policy for the next breaking change

### Evaluation Checklist

**Conceptual understanding:**

- [ ] Can explain why 202 Accepted fits background work
- [ ] Can explain at-least-once delivery and why idempotency follows
- [ ] Can explain why async does not speed up CPU work
- [ ] Can explain the outbox pattern and the problem it solves
- [ ] Can choose between offset and cursor pagination

**Design:**

- [ ] Jobs carry identifiers, not payloads
- [ ] Every retry policy is bounded
- [ ] Streams have explicit terminators
- [ ] No blocking calls remain on the event loop

---

# RESOURCES FOR DEEPER LEARNING

**Concepts:**

- "Designing Data-Intensive Applications" (message queues, stream processing, idempotency)
- Microservices.io articles on the transactional outbox pattern
- Stripe's engineering writing on idempotency keys

**Standards:**

- RFC 9110 (HTTP semantics, including 202 Accepted)
- NDJSON specification (ndjson.org)

**Documentation to consult:**

- FastAPI documentation on background tasks, lifespan events, and concurrency
- Asyncio documentation on tasks, cancellation, and timeouts
- Your chosen task queue's documentation on acknowledgement and retry settings

---

**End of Week 10 Content**

The main skill this week is deciding where work belongs. Work that the client needs now stays in the request. Work that must happen reliably, later, and possibly more than once goes into a system designed for exactly that.

**Next week:** Testing, monitoring and performance, or do you want to revisit any Week 10 concept first?