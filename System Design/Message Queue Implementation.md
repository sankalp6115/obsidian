# Production-Grade Asynchronous Message Queue: Architecture & Engineering Guide

## 1. Executive Summary & Core Philosophy

This document details the engineering design, evolution, and architecture of **PyQueue**—a persistent, decoupled asynchronous task processing system developed using **FastAPI**, **SQLite / SQLAlchemy**, and standalone **Worker processes**.

```
 +------------------------------------------------+
 |               CLIENT INTERFACES                |
 |   (HTML/JS Dashboard, REST API Clients, CLI)   |
 +-----------------------+------------------------+
                         |
   1. Enqueue Job (HTTP) | 4. Poll Status / Result
                         v
 +------------------------------------------------+
 |              FASTAPI PRODUCER API              |
 |    (Validation, Storage, File Management)      |
 +-----------------------+------------------------+
                         |
   2. Write Job State    | Read State
                         v
 +------------------------------------------------+
 |             SHARED STORAGE QUEUE               |
 |     (SQLite `jobs.db` -> Future: Redis)        |
 |   [pending] -> [running] -> [done / failed]    |
 +-----------------------+------------------------+
                         |
   3. Atomic Claim       | 5. Update Status & Result
                         v
 +------------------------------------------------+
 |            AUTONOMOUS WORKER FLEET             |
 |   Worker 1   |   Worker 2   |   Worker N       |
 |   (Sleep)    |  (Calculate) |   (Images)       |
 +------------------------------------------------+
```

### The Problem It Solves
In standard synchronous web architectures, long-running operations—such as heavy mathematical computation, large image compression, or third-party I/O—block the web server's request-response cycle. This exhausts thread/worker pools, spikes HTTP latency, and causes client timeouts.

### The Decoupled Solution
PyQueue introduces strict **Producer-Consumer decoupling**:
1. **Producer (FastAPI)**: Validates input schemas, assigns a unique identifier (`job_id`), writes the job to the database with `status="pending"`, and immediately returns an HTTP `201 Created` acknowledgment.
2. **Queue (Persistent Storage)**: Acts as the decoupled buffer preserving job requests across system reboots or crashes.
3. **Consumer (Worker Fleet)**: Independent OS processes continuously poll for pending jobs, atomically claim ownership, execute handlers, and persist results or failure traces.

---

## 2. Architecture & Design Journey: Simple to Final Implementation

### Stage 1: The Naive Approach (In-Memory / Synchronous)
Initially, the instinct in simple web applications is to handle tasks inside the API route or rely on simple in-process background tasks:
- **Flaws**:
  - If the server restarts or crashes mid-task, all queued and running work is irrevocably lost.
  - Compute-heavy jobs (like Pillow image compression) saturate the web server's CPU cores, causing API degradation.
  - No horizontal scalability: you cannot scale workers independently of the API gateway.

### Stage 2: Database-Backed Queue with a Single Worker
To solve persistence, a centralized `jobs` table was introduced in SQLite via SQLAlchemy. The API persists incoming tasks, and a separate script (`worker.py`) queries `status == 'pending'`, executes the task, and marks it `'completed'`.
- **Flaws**:
  - A single worker process becomes a throughput bottleneck.
  - If multiple workers are launched, they read the same pending job simultaneously, causing duplicate executions and race conditions.
  - If a worker crashes mid-execution, the job remains stuck in `status == 'running'` forever (zombie jobs).

### Stage 3: The Production-Grade Multi-Worker Architecture (Final Implementation)
The system was re-architected with production safety guarantees:
1. **Atomic Conditional Claiming**: Eliminates race conditions so multiple workers safely process a shared queue.
2. **Stale Job Crash Recovery**: An automated sweeper detects tasks stuck in `running` beyond a lease timeout and resets them to `pending`.
3. **Strict Validation Pipeline**: Pydantic models with discriminators ensure invalid jobs are rejected before entering the database.
4. **Dynamic Image Processing Pipeline**: Supports direct multipart file uploads, storing source files in `assets/image/{task_id}.{format}` and generating multi-quality compressed variants into `assets/output_images/{task_id}_{quality}.{format}`.
5. **Decoupled Client Dashboard**: Single-page frontend implementing non-blocking auto-polling and status indicators.

---

## 3. Directory Layout & Modular Separation of Concerns

The project follows clean architecture principles, strictly separating models, business logic, ingestion routers, and background consumers:

```text
message_queue/
├── app/
│   ├── api/                     # Ingestion Layer (FastAPI Routers)
│   │   ├── jobs.py              # POST /jobs, POST /jobs/upload, GET /jobs, GET /jobs/{id}
│   │   └── router.py            # API prefix aggregation (/api)
│   ├── core/                    # Core Configuration & Logging
│   │   ├── config.py
│   │   └── logging.py
│   ├── database/                # Database Engine & Session Management
│   │   ├── database.py          # SQLAlchemy SQLite engine & sessionmaker
│   │   ├── session.py           # Dependency injection `get_db()` yield generator
│   │   └── database_reset.py    # Database maintenance utility
│   ├── models/                  # SQLAlchemy ORM Entities
│   │   └── model.py             # `Job` model definition (UUID, payload, status, result)
│   ├── schemas/                 # Strict Pydantic Data Contracts
│   │   └── schema.py            # Discriminated JobRequests, Payloads, JobResponse
│   ├── services/                # Business Logic & File Operations
│   │   └── job_service.py       # Enqueueing, file saving, deserialization, ORM conversion
│   └── workers/                 # Asynchronous Consumers
│       ├── handlers.py          # Sleep, calculate, image compression logic
│       ├── recovery.py          # Stale job sweeper (resets zombie running jobs)
│       ├── worker.py            # Continuous claim-and-execute polling loop
│       └── multiprocessing.sh   # Process supervisor running concurrent workers
├── assets/                      # File Storage Layer
│   ├── image/                   # Source uploaded images (task_id.format)
│   └── output_images/           # Generated variants (task_id_quality.format)
├── docker/                      # Containerization Support
│   └── Dockerfile               # Production container definition
├── frontend/                    # Presentation Layer
│   └── index.html               # Reactive job enqueue & polling monitor
├── jobs.db                      # SQLite Persistent Queue Storage
├── pyproject.toml               # Project metadata & uv dependencies
└── README.md                    # Project run & setup instructions
```

---

## 4. SQLite as a Queue: Trade-offs & The Migration to Redis

### Why SQLite for Learning & Prototyping?
- **Zero Infrastructure**: Runs as an embedded file (`jobs.db`) without provisioning network daemons like RabbitMQ or Kafka.
- **ACID Guarantees**: Transactions ensure that job creations and state transitions are durable on disk.
- **Inspectability**: Any standard SQL client (`sqlite3`, TablePlus, DBeaver) can inspect the queue state instantaneously.

### Why SQLite is NOT a Real Message Queue
While functional for single-node development, SQLite presents inherent architectural limits:
1. **Table/Database-Level Write Locking**: SQLite only permits one active writer at a time. As worker count or ingestion rate increases, workers hit `database is locked` errors (`sqlite3.OperationalError`).
2. **Busy Polling Overhead**: Workers must continuously issue `SELECT ... WHERE status = 'pending'` queries every second, consuming disk I/O and CPU even when the queue is idle.
3. **No Native Pub/Sub or Streaming**: Workers cannot block on a reactive notification channel; they must poll.

### Future Architecture: Redis & Redis Streams
In a high-throughput production environment, SQLite is replaced by **Redis**:

| Feature | SQLite Queue Implementation | Redis Implementation |
| :--- | :--- | :--- |
| **Notification Pattern** | Polling (`time.sleep(1)`) | Push / Blocking pop (`BRPOP` or `XREADGROUP BLOCK`) |
| **Throughput** | ~50 - 200 operations/sec | ~50,000 - 100,000 operations/sec |
| **Locking Overhead** | File-level read/write locks | In-memory atomic commands (`LPUSH`, `RPOPLPUSH`) |
| **Concurrency Model** | Optimistic atomic update queries | Redis Streams with Consumer Groups & ACKs |
| **Storage Persistence** | Permanent relational disk storage | In-memory with RDB snapshots / AOF logs |

#### Redis Streams Migration Design:
- **Enqueue**: FastAPI calls `XADD queue:jobs * type image payload <json>`.
- **Claim**: Workers join a consumer group (`XREADGROUP GROUP workers worker_1 BLOCK 2000 COUNT 1 STREAMS queue:jobs >`).
- **Ack & Visibility**: Worker processes job and sends `XACK queue:jobs workers <id>`. If a worker dies, `XPENDING` and `XCLAIM` reassign abandoned jobs to surviving workers.

---

## 5. Concurrency & Atomic Job Picking

### The Race Condition Problem
When running four parallel worker processes (`multiprocessing.sh`), multiple workers can execute `SELECT * FROM jobs WHERE status = 'pending' LIMIT 1` at the exact same millisecond. Both workers see the same candidate job, claim it, and process it twice. In an email or payment queue, this causes catastrophic duplicate side-effects.

### The Atomic Claim Solution
PyQueue solves this at the database engine level via **Conditional Atomic Updates**:

```python
def claim_next_job(db: Session) -> Job | None:
    # 1. Identify candidate job
    candidate = db.query(Job).filter(Job.status == "pending").first()
    if not candidate:
        return None

    # 2. Atomic Compare-And-Swap
    rows_updated = db.query(Job).filter(
        Job.id == candidate.id,
        Job.status == "pending"  # Crucial: only updates if STILL pending
    ).update({
        "status": "running"
    })

    db.commit()

    # 3. Only the worker that successfully flipped 1 row owns the execution lock
    if rows_updated == 1:
        return candidate
    else:
        # Another worker won the race; yield ownership
        return None
```

```
           Worker A                            Worker B
              │                                   │
              ├─────── SELECT (sees Job 1) ───────┤
              │                                   │
              ▼                                   ▼
    UPDATE WHERE id=1                   UPDATE WHERE id=1
     AND status='pending'                AND status='pending'
              │                                   │
              ▼                                   ▼
      (rows_updated = 1)                  (rows_updated = 0)
              │                                   │
      [Claim Successful!]                   [Race Lost!]
      Processes Job 1...                    Backs off to poll again
```

---

## 6. Fault Tolerance: Zombie Jobs & Retry Recovery

### The "Zombie Job" Vulnerability
If a worker crashes (out-of-memory error, power failure, `kill -9`, or hardware reboot) while executing a job, SQLite still has `status = "running"`. Since standard worker loops only query `status = "pending"`, the job will remain abandoned in `running` forever.

### The Stale Job Sweeper (`recovery.py`)
To ensure **at-least-once execution guarantees**, a recovery supervisor monitors running tasks:

```python
DEAD_JOB_RETRY_INTERVAL = int(os.getenv("DEAD_JOB_RETRY_INTERVAL", 600))  # 10 minutes

def recover_stale_jobs():
    db = SessionLocal()
    try:
        threshold = datetime.now(timezone.utc) - timedelta(seconds=DEAD_JOB_RETRY_INTERVAL)
        
        # Reset jobs that have been stuck in 'running' longer than lease duration
        stale_jobs = db.query(Job).filter(
            Job.status == "running",
            Job.updated_at <= threshold
        ).all()

        for job in stale_jobs:
            print(f"[RECOVERY] Resetting stale job {job.id} back to pending.")
            job.status = "pending"
            job.error = f"Recovered from dead worker after {DEAD_JOB_RETRY_INTERVAL}s timeout."

        db.commit()
    finally:
        db.close()
```

### Structured Error Capture
When a job handler fails during execution, errors are captured, sanitized, and stored:
```python
try:
    job_payload = json.loads(job.payload)
    result = task_executor(job, job_payload)
    job.status = "completed"
    job.result = result
except Exception as e:
    job.status = "failed"
    job.error = str(e)
finally:
    db.commit()
```

---

## 7. The Image Processing Job: File Pipelines & Polymorphic Validation

### End-to-End Image Workflow
1. **Ingestion (`POST /api/jobs/upload`)**:
   - Client sends multipart file (`UploadFile`).
   - Server validates extension against allowlist (`.png`, `.jpg`, `.jpeg`, `.webp`).
   - Server generates `task_id = uuid.uuid4()`.
   - File is streamed to disk: `assets/image/{task_id}.{format}`.
   - Job is committed to SQLite with payload `{"image_path": "{task_id}.{format}", "operation": "compress"}`.
   - API returns `201 Created` with job schema.

2. **Asynchronous Execution (`image_handler`)**:
   - Worker picks up job.
   - Loads image via Pillow (`PIL.Image.open`).
   - Converts color space to `RGB`.
   - Compresses into target qualities (`[2, 25, 50, 75]`).
   - Writes generated files to `assets/output_images/{task_id}_{quality}.{format}`.
   - Stores array of generated paths in `job.result`.

```
[Client Upload]
       │
       ▼
 assets/image/5b31df14-....png
       │
       ▼ (Background Worker Claims Job)
       ├─► assets/output_images/5b31df14-..._2.png   (Low Quality / Preview)
       ├─► assets/output_images/5b31df14-..._25.png  (Mobile Quality)
       ├─► assets/output_images/5b31df14-..._50.png  (Standard Quality)
       └─► assets/output_images/5b31df14-..._75.png  (High Quality)
```

### Key Debugging Milestone: Pydantic Polymorphic Validation
During initial testing of image jobs, calls to `GET /api/jobs` crashed with HTTP 500:
- **Root Cause**: `JobResponse` strictly expected `result: dict[str, Any] | None`. While `calculation` and `sleep` handlers returned JSON dictionaries (`{"result": "..."}`), `image_handler` returned a JSON array of image objects (`[{"quality": 2, "path": "..."}, ...]`).
- **Resolution**: Updated `JobResponse.result` to support both collections and mappings:
  ```python
  class JobResponse(BaseModel):
      model_config = ConfigDict(from_attributes=True)
      id: str
      type: str
      payload: dict[str, Any]
      status: Literal["pending", "running", "failed", "completed"]
      result: dict[str, Any] | list[Any] | None = None
      error: str | None = None
      created_at: datetime
      updated_at: datetime
  ```

---

## 8. Multi-Worker Scaling & Process Management

To utilize multi-core server processors, a process supervisor script was created ([`multiprocessing.sh`](file:///Users/sankalpomar/Documents/Coding/backend/message_queue/app/workers/multiprocessing.sh)):

```bash
#!/bin/bash
# Multi-worker supervisor launching 4 independent worker processes

trap 'echo "Stopping workers..."; kill $(jobs -p); exit' SIGINT SIGTERM

echo "Starting 4 background queue workers..."
for i in {1..4}; do
    python -m app.workers.worker &
    echo "Worker $i started with PID $!"
done

wait
```
- Each worker runs its own isolated Python interpreter instance.
- GIL (Global Interpreter Lock) contention is bypassed because each process has its own address space and GIL.
- Heavy image processing is distributed evenly across multiple CPU cores.

---

## 9. Containerization & Docker Architecture

To package the message queue for deployment, Docker Compose orchestrates the decoupled topology:

```yaml
version: '3.8'

services:
  api:
    build:
      context: .
      dockerfile: docker/Dockerfile
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000
    volumes:
      - queue_data:/app/data
      - queue_assets:/app/assets
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=sqlite:////app/data/jobs.db
      - IMG_DIR=/app/assets/image
      - IMG_OUTPUT_DIR=/app/assets/output_images

  worker:
    build:
      context: .
      dockerfile: docker/Dockerfile
    command: python -m app.workers.worker
    deploy:
      replicas: 3   # Easily scale worker pool horizontally
    volumes:
      - queue_data:/app/data
      - queue_assets:/app/assets
    environment:
      - DATABASE_URL=sqlite:////app/data/jobs.db
      - IMG_DIR=/app/assets/image
      - IMG_OUTPUT_DIR=/app/assets/output_images

volumes:
  queue_data:
  queue_assets:
```

---

## 10. Frontend Client Dashboard

The user interface in [`frontend/index.html`](file:///Users/sankalpomar/Documents/Coding/backend/message_queue/frontend/index.html) exemplifies client-server decoupling:
- **Zero Page Refresh**: Uses `onsubmit="event.preventDefault(); return false;"` and `e.stopPropagation()` to eliminate unwanted browser form navigation.
- **Asynchronous Feedback**: Submitting an image or calculation updates the UI immediately to `PENDING`.
- **Reactive Polling Loop**: The client issues lightweight `GET /api/jobs/{id}` requests once every second until the worker flips the status to `COMPLETED` or `FAILED`.
- **Inspection Utilities**: Provides buttons to query individual tasks by UUID or fetch the entire queue list for observability.

---

## 11. Engineering Summary & Lessons Learned

| Challenge | Initial Symptom | Engineering Solution |
| :--- | :--- | :--- |
| **Blocking API** | Long sleeps & PIL image processing timed out HTTP requests. | Background worker decoupled via persistent job queue. |
| **Worker Race Conditions** | Parallel workers claimed and processed the same jobs. | Atomic conditional `UPDATE ... WHERE id = :id AND status = 'pending'`. |
| **Zombie / Abandoned Tasks** | Worker crashes left tasks stranded in `running` status forever. | Heartbeat / Lease-timeout sweeper recovering tasks older than lease threshold. |
| **Schema Serialization** | Pydantic raised 500 validation error when deserializing image list results. | Polymorphic schema update: `result: dict[str, Any] \| list[Any] \| None`. |
| **Browser Form Reloader** | Submitting tasks triggered full-page browser refresh. | Inline `onsubmit` prevention, event stopping, and async fetch hooks. |

This architecture bridges the conceptual gap between simple script automation and production-grade distributed queue engines, providing a solid blueprint for future expansion into Redis, Kafka, or Celery.

