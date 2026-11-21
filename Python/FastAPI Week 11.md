# FastAPI Week 11: Testing, Monitoring & Performance - Concept-Focused Guide

**Duration:** 13 hours **Learning Style:** Principles and reasoning first, tool usage via documentation **Prerequisites:** Weeks 1-10 completed **Goal:** Know what to test and why, how to see what a running system is doing, how to define "fast enough," and how to find where time actually goes

---

## Table of Contents

1. [Testing Strategy and the Test Pyramid - 2 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#testing-strategy-and-the-test-pyramid---2-hours)
2. [Test Design Principles - 2.5 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#test-design-principles---25-hours)
3. [Observability: Logs, Metrics, Traces - 2.5 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#observability-logs-metrics-traces---25-hours)
4. [Service Level Objectives and Alerting - 2 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#service-level-objectives-and-alerting---2-hours)
5. [Performance Reasoning - 2 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#performance-reasoning---2-hours)
6. [Load Testing and Profiling - 2 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#load-testing-and-profiling---2-hours)

---

# TESTING STRATEGY AND THE TEST PYRAMID - 2 HOURS

## Part 1: Why Test

### Formal Definition: Test

**Test:** An executable specification that states an expected behavior and fails when the actual behavior differs.

```
A test does three things:
1. Documents intended behavior (readable by future maintainers)
2. Detects regressions (changes that break existing behavior)
3. Constrains design (code that is hard to test is often coupled too tightly)
```

### The Purpose of Tests Is Confidence

```
Coverage percentage measures which lines ran during tests.
It does NOT measure whether the tests check anything meaningful.

A test suite with 100% coverage and no assertions has zero confidence.
A test suite with 60% coverage that exercises every important path has real confidence.

Principle: tests should answer "would I notice if this broke?"
```

## Part 2: The Test Pyramid

```
           ▲  End-to-end: few, slow, realistic (full system, real services)
          ╱ ╲
         ╱   ╲ Integration: moderate, real database or real dependencies
        ╱     ╲
       ╱       ╲ Unit: many, fast, isolated logic
      ╱─────────╲

Each layer costs more to run and maintain than the one below.
Each layer catches different classes of bugs.
```

### Concept: Where Bugs Actually Live

```
Pure logic bugs (calculations, validation rules):     unit tests catch them cheaply
Wiring bugs (dependency injection, route mismatches,
  serialization, query construction):                 integration tests catch them
Environment bugs (config, deployment, network):       end-to-end or staging catches them

Most FastAPI bugs live in the wiring between layers.
That is why integration tests with the real framework carry more value here than
a large pile of heavily mocked unit tests.
```

## Part 3: The Testing Trophy Alternative

```
Some teams emphasize integration tests more heavily ("testing trophy").
Reasoning: unit tests against mocks can pass while the real system fails,
because the mocks encode assumptions that may be wrong.

Principle: mock at boundaries you do not control (third-party APIs, clocks),
and use real implementations for things you do control (your database, your code).
```

---

# TEST DESIGN PRINCIPLES - 2.5 HOURS

## Part 1: Test Isolation

### Formal Definition: Test Isolation

**Test isolation:** Each test runs independently, producing the same result regardless of execution order or what other tests did.

```
Violations:
- Test A creates user "alice"; test B assumes no users exist
- Test C depends on a global variable changed by test D
- Shared database state leaks between tests

Symptoms: tests pass alone, fail together; pass on your machine, fail in CI.
```

### Strategies for Isolation

```
Fresh database per test:       strongest isolation, slowest
Transaction rollback per test: each test runs in a transaction that is discarded;
                               fast and isolated (connects to Week 5 transactions)
Unique data per test:          generate unique identifiers; no cleanup needed
                               but state accumulates
```

**Caveat:** Tests that commit their own transactions, or that exercise code which opens separate connections, may escape rollback. Know which of your code paths do this.

## Part 2: Dependency Overrides

### Concept: Replacing Dependencies in Tests

```
FastAPI dependencies (Week 4) are the seam for testing.
A test can replace the real database session with a test session,
or replace an authentication dependency with a fixed test user.

Principle: design dependencies so that the seam exists.
Code that creates its own database connection inside a function body
cannot be redirected in tests.
```

Connect to Week 4: the dependency injection pattern was designed partly for this.

## Part 3: What to Assert

### Concept: Assert Behavior, Not Implementation

```
Brittle:  assert the SQL query was called with exact text
Robust:   assert that after the request, the post exists with the expected title
          and that the unauthorized user got 403 and no row changed

Principle: test observable outcomes that matter to callers and to data.
```

### Concept: Test Names as Specifications

```
Weak:    test_posts()
Strong:  a non-author cannot delete another user's post, and the post remains

A reader should understand the requirement from the name alone.
```

## Part 4: Test Categories Specific to APIs

```
Contract tests:     response shape and status codes match the documented schema
Authorization tests: every protected endpoint refuses each role it should refuse (Week 8)
Validation tests:   malformed input produces 422 with useful detail; boundaries are respected
Error tests:        failures produce the generic client message and logged detail (Week 8 policy)
Idempotency tests:  retried requests with the same key produce one side effect (Week 10)
Streaming tests:    truncated streams are detectable (Week 10)
Real-time tests:    subscriptions require authorization; reconnect resyncs (Week 9)
```

## Part 5: Property-Based Testing

### Formal Definition: Property-Based Testing

**Property-based testing:** Instead of specifying example inputs, you specify invariants that must hold for all inputs, and a tool generates many inputs to search for counterexamples.

```
Example properties:
- Serializing then deserializing a post returns an equal post
- Cursor pagination over any dataset visits every item exactly once
- Any sequence of valid operations leaves the cursor-to-item mapping consistent

Tools: Hypothesis (Python)
Strength: finds edge cases humans don't think of.
Weakness: requires thinking in invariants, which takes practice.
```

## Part 6: Test Data

```
Factories:  functions that build valid objects with sensible defaults
            and let tests override only what matters.
Fixtures:   reusable setup that test functions request by name.

Principle: a test should show the reader only the data that matters to its assertion.
Hidden setup makes tests hard to read. Excessive setup makes them slow.
```

## Part 7: Flaky Tests

### Formal Definition: Flaky Test

**Flaky test:** A test that sometimes passes and sometimes fails without any change to the code under test.

```
Common causes:
- Timing assumptions (sleep-based waiting, race conditions in real-time code)
- Shared state between tests (see isolation)
- Dependence on current time, random values, or ordering of unordered collections
- External services that are sometimes slow or unavailable

Policy: a flaky test is a bug in the test or the code. Quarantine it,
then fix or delete it. Ignoring flaky tests trains the team to ignore failures.
```

---

# OBSERVABILITY: LOGS, METRICS, TRACES - 2.5 HOURS

## Part 1: The Question Observability Answers

### Formal Definition: Observability

**Observability:** The property of a system that lets you understand its internal state from its external outputs, so you can answer questions you did not anticipate in advance.

```
Monitoring answers questions you planned for:
"Is the error rate above 1%?"

Observability answers questions you did not plan for:
"Why are requests from one region failing only for large uploads on Tuesday?"

The second requires rich, high-dimensional data, not only pre-built dashboards.
```

## Part 2: The Three Pillars

### Logs: Discrete Events

```
A log line records something that happened:
  "payment failed: card declined, user_id=7, request_id=abc123"

Good logs are:
- Structured (key-value or JSON, not free text), so they can be queried
- Carry a request or correlation ID that ties together one user action
- Written at appropriate levels (debug, info, warning, error)
- Free of secrets and unnecessary personal data (Week 8)

Cost: high volume is expensive to store and search.
Principle: log decisions and failures, not every step.
```

### Metrics: Aggregated Numbers Over Time

```
A metric is a number sampled over time:
  request count, error count, latency distribution, queue depth, memory use

Types:
- Counter: only increases (total requests)
- Gauge: goes up and down (current connections, queue depth)
- Histogram: distribution of values in buckets (request latency)

Metrics are cheap to store and fast to query. They answer "how much?" and "how often?"
but not "which specific request?".
```

### Traces: One Request Across Components

```
A trace follows a single request through every component it touches:

Request → API handler (12 ms)
          ├─ database query (3 ms)
          ├─ cache lookup (0.2 ms)
          └─ call to payment service (180 ms)
             └─ payment service DB query (150 ms)

Each step is a span. Spans nest and carry timing and attributes.
Traces answer "where did the time go for THIS request?"
```

### How the Pillars Work Together

```
Metrics:  "p99 latency rose at 14:05"          (detect)
Traces:   "slow requests spent 150 ms in ..."  (localize)
Logs:     "the payment provider returned 429"  (explain)

Start with metrics to notice, traces to locate, logs to understand.
```

## Part 3: Correlation and Context

### Concept: Propagating the Request ID

```
A request enters your API and gets an ID.
That ID appears in:
- Every log line produced while handling it
- Outgoing calls to other services (as a header)
- Background jobs enqueued by it (Week 10)
- The response to the client (so support can search for it)

Without propagation, a single user complaint becomes hours of log archaeology.
```

### Concept: OpenTelemetry

```
OpenTelemetry is a vendor-neutral standard for generating traces, metrics, and logs.
Instrument once, send to any compatible backend.

Principle: prefer standard instrumentation over vendor-specific agents,
so you can change backends without rewriting code.
Check current documentation for supported FastAPI and database instrumentation.
```

## Part 4: Health Checks

```
Liveness:   "Is the process alive and not wedged?"
            Failure → orchestrator restarts the process.
            Should be cheap and depend on nothing external.

Readiness:  "Can this instance accept traffic right now?"
            Checks database and critical dependencies.
            Failure → load balancer stops sending traffic, but does not restart.

Common mistake: making liveness depend on the database.
A database outage then restarts every instance, which makes recovery harder.
```

## Part 5: Methods for Choosing What to Measure

### RED Method (for request-driven services)

```
Rate:     requests per second
Errors:   failed requests per second
Duration: latency distribution

Measure these for every endpoint and every downstream dependency.
```

### USE Method (for resources)

```
Utilization: fraction of capacity in use (CPU, connection pool)
Saturation:  work waiting because the resource is busy (queue length)
Errors:      error events from the resource

Apply to: database connection pool, worker queues, memory, file descriptors.
```

---

# SERVICE LEVEL OBJECTIVES AND ALERTING - 2 HOURS

## Part 1: Vocabulary

```
SLI (Service Level Indicator): a measurement of service quality
    Example: fraction of requests that succeed within 300 ms

SLO (Service Level Objective): a target for an SLI over a time window
    Example: 99.5% of requests succeed within 300 ms, over 30 days

SLA (Service Level Agreement): a contract with consequences if missed
    Example: refunds or credits if availability falls below 99.9%

Error budget: the allowed failure implied by the SLO
    Example: 99.5% target leaves 0.5% of requests that may fail
```

**Principle:** SLOs are internal engineering targets. SLAs are external promises. Set SLOs stricter than SLAs.

## Part 2: Why Error Budgets Matter

```
Error budget remaining: deploy freely, take risks.
Error budget exhausted: slow down, prioritize reliability work.

This converts a vague argument ("is it safe to ship?") into a shared, measurable rule.
```

## Part 3: Alerting Principles

### Concept: Alert on Symptoms, Not Causes

```
Symptom alert: "users see errors above 1% for 5 minutes"
Cause alert:   "CPU above 80%"

Symptom alerts page humans about things users feel.
Cause alerts are useful for diagnosis and dashboards, but as pages they produce noise:
high CPU may be harmless.
```

### Concept: Alert Fatigue

```
An alert that fires often and needs no action trains people to ignore alerts.

Every page should be:
- Actionable (someone knows what to do)
- Urgent (waiting until morning is wrong)
- Rare enough to be taken seriously

Review alerts regularly; delete the ones nobody acts on.
```

### Concept: Burn Rate

```
Burn rate: how fast the error budget is being consumed.
A high burn rate over a short window pages immediately.
A low burn rate over a long window creates a ticket instead.

This reduces false alarms from brief spikes while catching sustained problems quickly.
```

---

# PERFORMANCE REASONING - 2 HOURS

## Part 1: Latency and Throughput

```
Latency:    time for one request to complete
Throughput: requests completed per unit time

They are related but not the same.
A system can have low latency and low throughput (one fast lane, few cars),
or high throughput and high latency (many lanes, each slow).
```

## Part 2: Percentiles, Not Averages

### Concept: Why Averages Mislead

```
Ten requests with latencies:
9 requests at 50 ms, 1 request at 2000 ms
Average: 245 ms
Median (p50): 50 ms
p99: 2000 ms

The average describes nobody's experience.
Percentiles describe the distribution: "99% of requests were faster than X".
```

### Concept: Tail Latency

```
p99 and p99.9 are the tail.
In a request that calls 10 services in sequence, each with a 1% chance of slowness,
the chance that at least one is slow is roughly 10%.
Tail latency compounds across dependencies.

Principle: optimize the tail when users experience multiple dependent calls.
```

## Part 3: Queueing Intuition

### Concept: Utilization and Waiting

```
As a resource approaches full utilization, waiting time grows very fast.
A server at 50% busy has short queues.
A server at 95% busy has long, unpredictable queues.

Consequence: running systems near full capacity causes latency spikes
even when average load looks fine.
Plan headroom deliberately.
```

### Little's Law

```
In a stable system:
  average number in system = arrival rate × average time in system

Example: 100 requests/second arriving, each taking 0.2 seconds
  average concurrent requests = 100 × 0.2 = 20

Use it to size connection pools and worker counts:
  if you need 20 concurrent database operations, the pool needs at least 20 connections.
```

Connect to Week 6: connection pool sizing is this calculation.

## Part 4: Amdahl's Law

### Concept: Limits of Parallelism

```
If a fraction P of work can be parallelized, and you speed up that part by factor S,
overall speedup is limited by the serial fraction (1 − P).

Example: if 10% of a request is serial, even infinite parallelism
cannot make it more than 10× faster.

Principle: before optimizing, find the serial bottleneck.
Making parallel parts faster gives diminishing returns.
```

## Part 5: The Performance Process

```
1. Define the target: "p95 under 300 ms at 200 requests/second"
2. Measure the current state under realistic load
3. Find where time goes (profiling, traces)
4. Change one thing
5. Measure again under the same conditions
6. Keep the change only if it helps the target

Anti-patterns:
- Optimizing code that is not on the critical path
- Benchmarking on a laptop with a different database than production
- Changing several things at once and not knowing what helped
- Trusting a single run; variance matters
```

### Concept: Know Where Cost Lives

```
Typical sources of latency in a FastAPI service, in rough order of impact:
1. Database round trips and query shape (N+1 from Week 6, missing indexes)
2. Outbound calls to other services, with no timeouts or with long ones
3. Blocking code on the event loop (Week 10)
4. Serialization of large responses
5. Framework and middleware overhead (usually the smallest)

Fix the expensive ones first. Framework overhead rarely decides the outcome.
```

---

# LOAD TESTING AND PROFILING - 2 HOURS

## Part 1: Load Testing

### Formal Definition: Load Test

**Load test:** An experiment that applies a controlled, realistic rate of requests to a system and measures its latency, throughput, and error behavior.

### Concepts That Make Load Tests Honest

```
Realistic workload:  mix of endpoints in production proportions, not one hot endpoint
Realistic data:      tables with production-like sizes; empty tables hide slow queries
Realistic environment: similar hardware, same database engine, same network topology
Open vs closed model:
  Closed: N virtual users, each waits for a response before sending the next.
          Slow responses reduce the load automatically (hides problems).
  Open:   requests arrive at a fixed rate regardless of responses.
          Shows what happens when load exceeds capacity (more realistic for public traffic).
```

### Coordinated Omission

```
Problem: a load generator that waits for each slow response before sending the next
request under-reports latency, because the slow periods produce fewer samples.

Correct approach: measure latency from the scheduled send time, not the actual send time.
Check whether your tool corrects for coordinated omission before trusting its percentiles.
```

### What to Look For

```
- The knee: the load level where latency begins to climb steeply
- Saturation: which resource runs out first (connections, CPU, memory, database)
- Error behavior: do errors appear gracefully (429, 503) or as timeouts and crashes?
- Recovery: after load drops, do latencies return to normal?
```

## Part 2: Profiling

### Formal Definition: Profiling

**Profiling:** Measuring where a program spends its time or memory, at the level of functions or lines.

### Types of Profiling

```
CPU profiling:     which functions consume processor time
                   (sampling profilers have low overhead; py-spy can attach to running processes)
Memory profiling:  which allocations grow over time, useful for leaks
I/O observation:   time spent waiting on databases and network;
                   often more important than CPU in web services
```

### Concept: Profile Production-Like Conditions

```
A profile of a single request in an idle development process
can mislead. Concurrency changes the picture:
- Contention for connections
- Event loop congestion
- Garbage collection under load

Profile under realistic load when the question is throughput.
```

### Concept: Sampling vs Tracing Profilers

```
Sampling: periodically records the call stack. Low overhead, approximate.
Tracing:  records every call. High overhead, exact, distorts timing.

For production-adjacent investigation, sampling is usually the right choice.
```

## Part 3: Connecting Measurement to Action

```
Symptom (from metrics):      p95 latency doubled after Tuesday's deploy
Localization (from traces):  the slow span is one database query
Diagnosis (from profiling):  the query lost its index, or N+1 returned
Verification (load test):    p95 returns to baseline under the same load

Every step in that chain uses a different tool.
Understanding what each tool can and cannot tell you is the real skill.
```

---

# KEY CONCEPTS SUMMARY

## Testing

- Tests are executable specifications; coverage does not measure their quality
- The pyramid balances speed against realism; most FastAPI bugs live in the wiring
- Isolate tests with rollback, fresh databases, or unique data
- Dependency overrides are the seam for replacing infrastructure in tests
- Test behavior and outcomes, not implementation details
- Flaky tests are bugs; quarantine and fix them

## Observability

- Logs record events, metrics record aggregates, traces follow one request
- Structured logs with propagated request IDs make incidents searchable
- Liveness must not depend on external systems
- RED for services, USE for resources

## SLOs and Alerting

- SLI measures, SLO targets, SLA promises, error budget is the allowance
- Alert on symptoms users feel, not every internal cause
- Burn rate separates urgent pages from tickets

## Performance

- Latency and throughput are different; use percentiles, not averages
- Tail latency compounds across dependent calls
- Utilization near full causes unpredictable queues
- Little's Law sizes pools; Amdahl's Law limits parallelism gains
- Measure, change one thing, measure again under the same conditions

## Load Testing and Profiling

- Use realistic workloads, data sizes, and environments
- Open-model load shows overload behavior; closed-model load can hide it
- Beware coordinated omission in latency measurements
- Profile under realistic concurrency; sampling profilers suit production-adjacent work

---

# WEEK 11 CHECKPOINT PROJECT

## Project: Test, Observe and Measure the Blog API

### Objective

Make the Week 10 Blog API testable, observable, and measured against explicit performance targets.

### Part 1: Test Strategy Document

- Describe your pyramid: how many unit, integration, and end-to-end tests, and why
- Identify the seams: which dependencies are overridden in tests, and which are real
- Explain your isolation strategy and where it might fail

### Part 2: Test Coverage of Risk

Map tests to the risk categories from earlier weeks:

- Authorization on every protected endpoint (Week 8)
- Validation boundaries (Week 3)
- Idempotent job handling (Week 10)
- Cursor pagination stability (Week 10)
- Real-time subscription authorization (Week 9)

For each, name the test and the behavior it protects.

### Part 3: Observability Design

- Define the structured log fields for every request
- Define the request ID propagation path, including into background jobs
- Define liveness and readiness checks, with the dependencies each one checks
- Choose RED metrics for each endpoint

### Part 4: SLOs

- Choose one SLI and one SLO for the post feed (for example, availability and latency)
- Calculate the error budget over 30 days
- Write one symptom-based alert and one burn-rate alert

### Part 5: Performance Investigation

Describe a hypothetical investigation:

- A symptom from metrics
- The trace that localizes it
- The profile or query plan that diagnoses it
- The load test that verifies the fix

Use Little's Law to size the database connection pool for a stated target throughput.

### Part 6: Load Test Plan

- Define the workload mix across endpoints
- Define the data set size
- State whether your load model is open or closed, and why
- Define what counts as passing: target percentiles and error rates
- Identify the expected knee and the first resource you expect to saturate

### Evaluation Checklist

**Conceptual understanding:**

- [ ] Can explain why coverage alone is not confidence
- [ ] Can explain logs, metrics, and traces, and when each answers a question
- [ ] Can explain why averages mislead and percentiles help
- [ ] Can explain Little's Law and apply it to pool sizing
- [ ] Can explain the difference between open and closed load models

**Practice:**

- [ ] Tests are isolated and repeatable
- [ ] Every request carries a request ID into logs
- [ ] Health checks are split into liveness and readiness
- [ ] One SLO and its alerts are defined
- [ ] A load test plan states its pass criteria

---

# RESOURCES FOR DEEPER LEARNING

**Testing:**

- pytest documentation (fixtures, parametrization, dependency patterns)
- FastAPI documentation on testing and dependency overrides
- Hypothesis documentation (property-based testing)

**Observability:**

- OpenTelemetry documentation
- Google SRE Book, chapters on monitoring and service level objectives (free online)
- "Observability Engineering" (Majors, Fong-Jones, Miranda)

**Performance:**

- "Systems Performance" by Brendan Gregg (methods, USE, profiling)
- Brendan Gregg's website on flame graphs and profiling
- Documentation for your load testing tool (Locust, k6) on open vs closed models

---

**End of Week 11 Content**

Testing tells you whether the system does what you intended. Observability tells you what it is doing now. Performance work tells you where time goes. Each answers a different question, and mature systems need all three.

**Next week:** Deployment and advanced operations, the final week. Or do you want to revisit any Week 11 concept first?