# FastAPI Week 12: Deployment & Advanced Operations - Concept-Focused Guide

**Duration:** 13 hours **Learning Style:** Operating models and failure reasoning first, tool usage via documentation **Prerequisites:** Weeks 1-11 completed **Goal:** Understand how a FastAPI service moves from a laptop to production, how it stays up through change, and how it recovers when things go wrong

---

## Table of Contents

1. [What "Production" Means - 1.5 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#what-production-means---15-hours)
2. [Configuration and the Twelve-Factor Principles - 1.5 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#configuration-and-the-twelve-factor-principles---15-hours)
3. [Process Models and ASGI Servers - 2 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#process-models-and-asgi-servers---2-hours)
4. [Containers and Packaging - 2 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#containers-and-packaging---2-hours)
5. [Edge: Reverse Proxies and TLS - 1.5 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#edge-reverse-proxies-and-tls---15-hours)
6. [Releases: Zero-Downtime Deployment - 2 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#releases-zero-downtime-deployment---2-hours)
7. [Migrations in Production - 1.5 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#migrations-in-production---15-hours)
8. [CI/CD and Supply Chain - 1 Hour](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#cicd-and-supply-chain---1-hour)
9. [Backups, Disaster Recovery and Operating Over Time - 1.5 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#backups-disaster-recovery-and-operating-over-time---15-hours)

---

# WHAT "PRODUCTION" MEANS - 1.5 HOURS

## Part 1: The Gap Between Working and Operable

### Formal Definition: Operable Service

**Operable service:** A service that can be deployed, configured, observed, scaled, updated, and recovered by people who did not write it, without the author present.

```
"It works on my machine" tests functionality.
"It runs in production" tests operability.

Week 11 gave you observability and tests. Week 12 puts them to work
in an environment you do not fully control.
```

## Part 2: Environments

```
Development:  your laptop; fast feedback; fake data; relaxed settings
Staging:      mirrors production configuration and topology; real-like data volume
Production:   real users; real data; strict controls

Principle: the closer staging is to production, the more its results mean.
Differences in engine version, network path, or configuration are where
"passed in staging, failed in production" comes from.
```

## Part 3: Reliability Vocabulary for Operations

```
RPO (Recovery Point Objective): how much data loss is acceptable, measured in time
    Example: at most 15 minutes of writes lost
RTO (Recovery Time Objective): how long recovery may take
    Example: service restored within 1 hour

These are business decisions, not technical ones.
Engineers make them achievable and make the cost visible.
```

---

# CONFIGURATION AND THE TWELVE-FACTOR PRINCIPLES - 1.5 HOURS

## Part 1: Config Is What Changes Between Environments

### Formal Definition: Configuration

**Configuration:** Values that differ between deployments of the same code, such as database URLs, API keys, feature flags, and log levels.

```
Same code, different configuration:
- Development: database on localhost, debug logging, no real email
- Production:  managed database, warning-level logging, real email provider
```

## Part 2: Principles

### Concept: Store Config in the Environment

```
The twelve-factor app methodology recommends storing configuration
in environment variables rather than in code or committed files.

Reasons:
- Code can be deployed to many environments without modification
- Secrets stay out of version control (Week 8)
- Changing config does not require a new build
```

### Concept: Strict Separation of Config and Code

```
Test: could you open-source the repository right now without leaking anything?
If not, something belongs in the environment instead.

Exception: non-secret defaults that make the code work out of the box.
```

### Concept: Fail Fast on Bad Config

```
A service that starts with a missing database URL and fails on the first request
is harder to diagnose than one that refuses to start with a clear message.

Validate configuration at startup, once, with a typed settings object.
Connects to Week 2: Pydantic validation applies to settings as well as request bodies.
```

## Part 3: Development/Production Parity

```
Keep these as similar as practical:
- Database engine and version
- Operating system and runtime version
- Dependency versions (pinned)
- Backing services (cache, queue, storage type)

Differences cost debugging time later.
```

---

# PROCESS MODELS AND ASGI SERVERS - 2 HOURS

## Part 1: What Runs Your App

### Formal Definition: ASGI Server

**ASGI server:** A program that listens for network connections, parses HTTP or WebSocket traffic, and calls your ASGI application for each request. Uvicorn is the common choice for FastAPI.

```
Client → ASGI server (listens on a port) → your FastAPI app → response

The ASGI server handles: sockets, HTTP parsing, connection lifecycle.
Your app handles: routing, logic, responses.
```

**Connection to Week 9:** The WebSocket handshake and upgrade are handled by the server layer. Your server configuration determines whether WebSockets are supported at all.

## Part 2: Concurrency at the Process Level

### Concept: Workers

```
A single process runs one event loop.
Work that blocks the loop (Week 10) stalls the whole process.
CPU-heavy work uses one core.

To use multiple cores, run multiple worker processes.
Each worker has its own event loop, memory, and connection pool.
```

### Concept: Worker Count

```
Too few workers: cores idle, requests queue.
Too many workers: memory duplicated per process, connection pools multiply,
                  database connection limits exhausted.

Connection budget example (Week 11, Little's Law and pool sizing):
  workers × pool size per worker ≤ database connection limit (minus headroom)

Always do this arithmetic before raising worker counts.
```

### Concept: Process Managers

```
A process manager supervises workers: restarts crashed ones, handles signals,
and coordinates graceful restarts.

Common pattern: a process manager supervising several ASGI worker processes.
Alternatively, containers with one process each, and an orchestrator handling restarts.

Principle: let something whose job is supervision handle supervision.
```

## Part 3: Graceful Shutdown

### Concept: What Happens When a Process Stops

```
Stop signal arrives (SIGTERM).
Correct behavior:
1. Stop accepting new connections
2. Finish in-flight requests within a grace period
3. Close WebSockets with a "going away" code (Week 9)
4. Flush logs and release resources
5. Exit

Incorrect behavior: exit immediately, dropping requests mid-flight.
```

### Concept: Grace Periods Must Be Consistent

```
Three timeouts must agree:
- Application shutdown timeout
- Process manager or orchestrator kill timeout
- Longest legitimate request duration

If the orchestrator kills the process before the grace period ends,
graceful shutdown never happens. Check the orchestration documentation
for how these interact.
```

---

# CONTAINERS AND PACKAGING - 2 HOURS

## Part 1: Why Containers

### Formal Definition: Container

**Container:** A packaged, isolated process environment that includes the application and its dependencies, runnable identically wherever a container runtime exists.

```
Without containers:  "install Python 3.12, these system libraries, this pip version..."
With containers:     "run this image"

The image is a snapshot. The same image runs in development, staging, and production.
```

## Part 2: Image Concepts

### Concept: Images Are Layers

```
Each instruction in an image build creates a layer.
Layers are cached. Changes near the top invalidate everything below.

Practical consequence: copy dependency manifests and install dependencies
before copying application source. Source changes often; dependencies change rarely.
```

### Concept: Multi-Stage Builds

```
Build stage:   has compilers, build tools, and dev dependencies
Runtime stage: has only the runtime and the installed application

Result: smaller images with less attack surface.
Build tools should not exist in production images.
```

### Concept: Least Privilege in Containers

```
- Run as a non-root user
- Read-only filesystem where possible
- No secrets baked into the image (secrets come from the environment at runtime)
- Expose only the port the application needs

An image is a build artifact that may be stored, copied, and shared.
Anything baked into it is effectively published to everyone with access.
```

## Part 3: Dependency Pinning

```
Unpinned:  fastapi>=0.100       → a rebuild next month may install a different version
Pinned:    fastapi==x.y.z       → reproducible builds

Use a lock file so every build installs the same transitive dependencies.
Update dependencies deliberately, with tests, not implicitly on rebuild.
```

---

# EDGE: REVERSE PROXIES AND TLS - 1.5 HOURS

## Part 1: Why a Proxy in Front

### Formal Definition: Reverse Proxy

**Reverse proxy:** A server that receives client requests on behalf of backend servers and forwards them, often adding TLS, caching, compression, and routing.

```
Client → reverse proxy (TLS, rate limits, static files, routing) → ASGI workers

Responsibilities usually delegated to the proxy:
- TLS termination (certificates, protocol versions)
- Request size and timeout limits at the edge
- Compression
- Serving static assets
- Routing to multiple services by path or host
```

**Principle:** Application servers should not be exposed directly to the internet. The proxy is the hardened boundary.

## Part 2: Trust and Forwarded Headers

### Concept: Who Is the Client?

```
Behind a proxy, the application sees the proxy's IP address.
Proxies add headers such as X-Forwarded-For and X-Forwarded-Proto
to convey the original client address and scheme.

Danger: if the application trusts these headers from ANY source,
clients can forge their own IP address, defeating IP-based rate limits (Week 8).

Rule: trust forwarded headers only from the proxy's address.
Configure the application's trusted-proxy list explicitly.
```

## Part 3: TLS Concepts

```
Certificate:  proves the server's identity for a domain
Termination:  where TLS ends; at the proxy, the application speaks plain HTTP internally
Renewal:      certificates expire; automate renewal, or outages follow
HSTS:         (Week 4) tells browsers to use HTTPS only

Internal traffic: decide whether proxy-to-application traffic must be encrypted.
Inside a trusted private network, termination at the edge is common.
Across untrusted networks, encrypt the hop as well.
```

## Part 4: Timeouts at Every Layer

```
Each layer has its own timeouts:
- Client
- Proxy (upstream and read timeouts)
- Application (request handler, database, outgoing calls)
- Database (statement timeout)

If the proxy gives up at 30 seconds while the application keeps working for 60,
the work is wasted and the user sees an error.

Principle: timeouts should decrease as you move inward,
so the innermost layer gives up first and reports the failure.
```

---

# RELEASES: ZERO-DOWNTIME DEPLOYMENT - 2 HOURS

## Part 1: The Problem

```
Naive deployment: stop the old version, start the new one.
Result: a window of downtime, dropped in-flight requests,
and disconnected WebSocket clients (Week 9).

Goal: move users from old to new without them noticing.
```

## Part 2: Strategies

### Rolling Deployment

```
Replace instances one at a time (or a few at a time).
Old and new versions run side by side during the rollout.

Pros: simple, no duplicate capacity needed.
Cons: two versions coexist; APIs and database schemas must be compatible with both.
```

### Blue-Green Deployment

```
Two full environments: blue (live) and green (idle).
Deploy to green, test, switch the router to green.
Rollback: switch back to blue.

Pros: instant switch and instant rollback.
Cons: double capacity during deployment; database schema changes must still be compatible.
```

### Canary Deployment

```
Send a small fraction of traffic (say 5%) to the new version.
Watch metrics (Week 11: error rate, latency). Increase if healthy; roll back if not.

Pros: limits blast radius of a bad release.
Cons: needs good metrics and the ability to route by percentage.
```

## Part 3: Compatibility Is the Real Constraint

### Concept: Mixed-Version Operation

```
During any rollout, two versions handle traffic simultaneously.
So:
- The new version must accept requests the old version's clients send
- The old version must tolerate data written by the new version
- Background jobs enqueued by one version must be processed by the other (Week 10)

Principle: deploy changes in backward-compatible steps.
```

### Concept: Expand and Contract

```
Renaming a database column in one step breaks the old version.
Safe sequence:
1. Expand:   add the new column; write to both; keep reading the old one
2. Migrate:  backfill existing rows
3. Switch:   read from the new column
4. Contract: remove the old column once no version uses it

Each step is deployable and reversible on its own.
```

## Part 4: Health Checks in Deployment

```
Readiness (Week 11) gates traffic:
- A new instance receives traffic only after it reports ready.
- A failing new instance is never added to the pool.

Liveness triggers restarts, not rollouts.
Confusing the two can cause restart loops during a bad deploy.
```

## Part 5: Rollback

```
Rollback is only as fast as its prerequisites:
- Old image still available
- Database changes compatible with the old version (expand/contract)
- Configuration changes reversible

Principle: decide the rollback plan BEFORE deploying.
A rollback that requires a schema change you cannot reverse is not a rollback.
```

---

# MIGRATIONS IN PRODUCTION - 1.5 HOURS

## Part 1: Connecting to Week 5

```
Week 5 introduced Alembic migrations for development.
Production adds constraints:
- Tables are large; some operations lock them for long periods
- Traffic continues during migration
- Rollback may be impossible if data is destroyed
```

## Part 2: Migration Risk Concepts

### Concept: Locking

```
Some schema changes take locks that block reads or writes.
On a large table, a lock held for minutes is an outage.

Before running a migration in production, know:
- Which lock it takes
- How long it runs on production-sized data
- Whether it can be done online (without blocking)
```

### Concept: Destructive Changes

```
Dropping a column or table is irreversible without a backup.
Renaming looks harmless but breaks any code still using the old name.

Principle: destructive changes are always the last step of a multi-release sequence,
performed after the backup has been verified.
```

### Concept: Migration Ordering with Deploys

```
Option A: migrate first, then deploy code
  Works if the schema change is backward compatible with the old code.

Option B: deploy code first, then migrate
  Works if the new code tolerates the old schema.

Expand/contract makes both orders safe.
Decide the order per change, and write it in the runbook.
```

### Concept: Migrations Should Run Once

```
If every application instance runs migrations at startup,
concurrent instances race each other.

Run migrations as a separate, single step in the deployment pipeline.
```

---

# CI/CD AND SUPPLY CHAIN - 1 HOUR

## Part 1: Continuous Integration

### Formal Definition: Continuous Integration

**CI:** Automatically building and testing every change, so integration problems surface within minutes rather than at release time.

```
A change should trigger:
- Dependency installation from the lock file
- Linting and type checking
- Unit and integration tests (Week 11)
- Security scanning of dependencies and images
- Building the image once, tagged by commit

Principle: build the artifact once, then promote the same artifact through environments.
Rebuilding per environment means you never tested what you deploy.
```

## Part 2: Continuous Delivery

```
Continuous delivery: every change that passes CI is deployable.
Continuous deployment: every change that passes CI is deployed automatically.

Choose based on risk tolerance and the strength of your tests and metrics.
Automated deployment without good health gating amplifies mistakes.
```

## Part 3: Supply Chain Security

### Concept: Your Dependencies Are Code You Run

```
Every package you install executes with your application's privileges.
Risks:
- Known vulnerabilities in dependencies
- Typosquatting (a malicious package with a similar name)
- Compromised maintainer accounts

Mitigations:
- Pin versions and use lock files
- Scan for known vulnerabilities in CI
- Review new dependencies before adding them
- Minimize dependency count where practical
```

### Concept: Provenance

```
Ideally you can answer: which commit built this image, from which dependencies,
by which pipeline, and who approved it?

Build provenance and signed artifacts make this answerable.
Check your platform's documentation for supported attestation features.
```

---

# BACKUPS, DISASTER RECOVERY AND OPERATING OVER TIME - 1.5 HOURS

## Part 1: Backups

### Concept: A Backup You Have Not Restored Is a Hope

```
Backups need three properties:
1. They happen on schedule (automated, monitored)
2. They are stored separately from the system they protect
3. They are restorable (tested regularly)

Most organizations discover broken backups during an incident.
Schedule restore drills and time them against the RTO.
```

### Concept: Point-in-Time Recovery

```
Full backups plus transaction logs allow restoring to any moment,
which is how RPO values below one hour are achieved.
Your database platform's documentation describes what it supports.
```

### Concept: What Else Needs Backing Up

```
- Database (obviously)
- Uploaded files and object storage (Week 3)
- Configuration and secrets (in a secret manager with its own recovery plan)
- Infrastructure definitions (in version control)
- Queue contents, if messages cannot be regenerated (Week 10)
```

## Part 2: Disaster Recovery

### Concept: Failure Domains

```
A failure domain is a set of things that fail together:
- One process
- One machine
- One availability zone or data center
- One region
- One provider account

Redundancy only helps across domains that fail independently.
Two replicas on one machine survive a process crash but not a machine failure.
```

### Concept: Runbooks

```
A runbook is a written procedure for a known failure.
Good runbooks include:
- Symptoms (how you know this is happening)
- Immediate actions (stop the bleeding)
- Diagnosis steps
- Recovery steps
- Verification (how you know it is fixed)
- Who to contact

Write runbooks for failures you have seen, and for failures you consider likely.
Test them before you need them.
```

### Concept: Incident Response and Review

```
During an incident: restore service first, understand cause second.
After an incident: a blameless review asks what the system allowed to happen,
and what would have made it safer or faster to recover.

Output: concrete action items, tracked to completion.
```

## Part 3: Capacity and Cost Over Time

```
Plan for growth:
- Watch trends in metrics (Week 11), not only current values
- Scale stateless services horizontally (Week 9 showed why state is the hard part)
- Scale the database deliberately; it is usually the hardest tier to change
- Review cost regularly; unused resources accumulate

Principle: services that hold no local state are easy to scale, replace, and recover.
Keep state in a small number of well-understood places.
```

## Part 4: Operating Responsibly

```
Least privilege for runtime: the application's database user should not own the schema
or have rights it never uses. Week 8 applies to infrastructure too.

Access review: periodically remove access that is no longer needed.
Audit: know who deployed what, and when (Week 8 audit logging).
```

---

# KEY CONCEPTS SUMMARY

## Production Readiness

- Operable means others can deploy, observe, and recover it
- Staging should mirror production; differences hide failures
- RPO and RTO are business targets engineering must meet

## Configuration

- Config in the environment, secrets outside the image
- Validate configuration at startup and fail fast
- Keep dev and production parity in engines, versions, and services

## Serving

- ASGI servers handle sockets; your app handles logic
- Workers multiply cores and memory; budget database connections per worker
- Graceful shutdown finishes in-flight work within a consistent grace period

## Packaging

- Images are layered; order instructions for cache efficiency
- Multi-stage builds keep build tools out of production
- Run as non-root, pin dependencies, keep secrets out of images

## Edge

- Reverse proxies terminate TLS and absorb edge concerns
- Trust forwarded headers only from known proxies
- Timeouts should shrink as you move inward

## Releases

- Rolling, blue-green, and canary trade capacity, speed, and blast radius
- Every release is a mixed-version period; stay backward compatible
- Expand and contract makes schema changes safe and reversible
- Decide the rollback plan before deploying

## Migrations

- Watch locks, data size, and destructive operations
- Choose migration order per change; run migrations once, as a separate step

## Pipeline and Supply Chain

- Build once, promote the same artifact
- Dependencies are code you run; pin, scan, and review them

## Operations

- A backup you have not restored is a hope
- Failure domains determine what redundancy actually protects against
- Runbooks turn known failures into repeatable recovery
- Blameless reviews turn incidents into system improvements

---

# WEEK 12 CHECKPOINT PROJECT: FINAL CAPSTONE

## Project: Deploy and Operate the Blog API

### Objective

Take the complete Blog API built across Weeks 1-11 and specify how it would run in production, including what happens when it fails.

### Part 1: Deployment Architecture

Draw the topology:

- Client, edge proxy, ASGI workers, database, cache, queue, workers, object storage
- Label each component's failure domain
- State where TLS terminates and where trusted proxy headers are accepted

### Part 2: Configuration Inventory

List every configuration value and secret, marking each as:

- Environment variable, secret manager, or build-time constant
- Which values differ between development, staging, and production

### Part 3: Container Specification

- Describe your multi-stage build and what the runtime stage contains
- State how dependencies are pinned
- State the user the process runs as and the port it exposes

### Part 4: Release Strategy

- Choose rolling, blue-green, or canary, and justify the choice
- Walk through one schema change using expand and contract
- Write the rollback plan for a bad release, including which steps are reversible

### Part 5: Pipeline

Describe the CI/CD stages for one commit, from test to production, noting:

- Where the image is built and tagged
- Where the migration runs
- Which gates must pass before promotion

### Part 6: Reliability and Recovery

- Define RPO and RTO for the Blog API, with justification
- Describe your backup schedule, storage location, and restore drill
- Write a runbook for one scenario: the database becomes unavailable
- Write a runbook for one scenario: a release causes a spike in 500 errors

### Part 7: Operations Review

Write a short blameless review of a hypothetical incident:

- Timeline
- What the system allowed to happen
- Action items with owners and due dates

### Evaluation Checklist

**Conceptual understanding:**

- [ ] Can explain the difference between liveness and readiness in deployment
- [ ] Can explain why mixed-version operation requires backward compatibility
- [ ] Can explain expand and contract
- [ ] Can explain failure domains and why redundancy must cross them
- [ ] Can explain why the image should be built once and promoted

**Practice:**

- [ ] Configuration comes from the environment and is validated at startup
- [ ] Container runs as non-root with pinned dependencies
- [ ] Proxy trusts forwarded headers only from known addresses
- [ ] Rollback plan exists before the release
- [ ] Backups have a tested restore procedure

---

# PROGRAM COMPLETION: THE FULL ARC

## What You Have Covered Across Twelve Weeks

```
Phase 1 (Weeks 1-2):   Python foundations, FastAPI fundamentals
Phase 2 (Weeks 3-4):   Advanced routing, validation, dependencies, middleware
Phase 3 (Weeks 5-6):   Relational databases, ORM, migrations, optimization
Phase 4 (Weeks 7-8):   Tokens, OAuth2, authorization models, threat thinking
Phase 5 (Weeks 9-10):  Real-time communication, background work, streaming
Phase 6 (Weeks 11-12): Testing, observability, performance, deployment, operations
```

## The Thread Through All Twelve Weeks

```
Week 1-4:   How code is structured and how requests flow
Week 5-6:   How data is stored and how queries cost time
Week 7-8:   How identity is proven and how access is decided
Week 9-10:  How time and failure shape communication and work
Week 11-12: How a system is verified, observed, and kept running
```

## Suggested Next Steps

```
- Build the capstone end to end, even on a small scale
- Read the official documentation for each tool referenced here, in the version you use
- Revisit Week 8 threat modeling against your own deployed API
- Pick one area (performance, security, or operations) to go deeper in
```

---

# RESOURCES FOR DEEPER LEARNING

**Operations and reliability:**

- Google SRE Book (free online) and "The Site Reliability Workbook"
- "Designing Data-Intensive Applications" (replication, partitioning, failure)
- "Release It!" by Michael Nygard (stability patterns)

**Deployment:**

- The Twelve-Factor App methodology (12factor.net)
- Docker and your container runtime's documentation on multi-stage builds
- Uvicorn and Gunicorn documentation on workers and graceful shutdown
- Your reverse proxy's documentation on forwarded headers and timeouts

**Security in operations:**

- OWASP Docker Security Cheat Sheet
- OWASP Cheat Sheet Series index (for the broader catalog)

---

**End of Week 12 and the Program**

Twelve weeks ago the goal was to write an API that works. The goal now is a service that others can deploy, observe, secure, and recover without you in the room. Keep the habit that carried through every week: state the failure you are designing for, then design for it.

Congratulations on completing the program.