**Duration:** 13 hours  
**Learning Style:** Concepts first, principles over code  
**Prerequisites:** Week 1-5 completed  
**Goal:** Master advanced database patterns, optimization principles, and architectural decisions

---

## Table of Contents

1. [SQLModel Philosophy & Design - 2.5 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#sqlmodel-philosophy--design---25-hours)
2. [Query Optimization & N+1 Problems - 3 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#query-optimization--n1-problems---3-hours)
3. [Performance Fundamentals - 2.5 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#performance-fundamentals---25-hours)
4. [Caching Patterns & Strategies - 2.5 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#caching-patterns--strategies---25-hours)
5. [Testing with Databases - 2 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#testing-with-databases---2-hours)

---

# SQLMODEL PHILOSOPHY & DESIGN - 2.5 HOURS

## Part 1: The Problem SQLModel Solves

### Formal Definition: Code Duplication Problem

**The Duplication Challenge:** In current approaches (SQLAlchemy ORM + Pydantic), you define data structure twice:

1. **SQLAlchemy Model** - Defines database schema
2. **Pydantic Schema** - Defines request/response validation

This violates DRY principle and creates maintenance burden.

### The Actual Problem (Why It Matters)

```
Scenario: Add a new field to users (phone number)

Week 1-5 approach:
1. Modify SQLAlchemy User model
2. Modify UserCreate schema  
3. Modify UserResponse schema
4. Create Alembic migration
5. Ensure validation constraints match in both
6. Update documentation in both places
7. Risk: Constraints inconsistent between DB and API

Result: Same validation logic defined in 2+ places
        Risk of inconsistency grows with codebase size
```

### Why Duplication Is Dangerous

**Concept: Single Source of Truth**

When the same information exists in multiple places:

- Changes in one place get forgotten in others
- Bugs arise from inconsistencies
- Maintenance becomes harder
- Testing must cover both places

Example of inconsistency:

```
SQLAlchemy: email = Column(String(100))
Pydantic: email = Column(String) with regex validation

API accepts: 150-character email
Database rejects: field too long
User gets error that doesn't match validation
```

---

## Part 2: SQLModel Solution

### Formal Definition: SQLModel

**SQLModel:** A library that combines Pydantic and SQLAlchemy into single model definition. One model works as:

- Database ORM model
- Pydantic validation schema
- Request body validator
- Response serializer

### Conceptual Architecture

```
Traditional Approach:
┌─────────────────────┐
│  SQLAlchemy Model   │ (Database)
└─────────────────────┘
         ↕ Manual sync
         ↕ (Error-prone)
┌─────────────────────┐
│   Pydantic Schema   │ (API)
└─────────────────────┘

SQLModel Approach:
┌─────────────────────┐
│    SQLModel         │
│  (Single Model)     │
│  Validates → ✓      │
│  Stores → ✓         │
│  Serializes → ✓     │
└─────────────────────┘
        Single
      Source of
       Truth
```

### Key Philosophical Differences

**Concept 1: Type System as Contract**

In SQLModel, the Python type system IS the contract:

- Type hint tells Pydantic how to validate
- Same type hint tells SQLAlchemy how to store
- One place to define, everywhere it applies

```
field: str = Field(..., min_length=1, max_length=100)

This single definition:
- Validates API input (Pydantic)
- Stores in database (SQLAlchemy)  
- Documents API (OpenAPI)
- Protects database (constraints)
```

**Concept 2: Model as API Contract**

SQLModel treats your model as the API contract:

- No separate request/response schemas
- What you store is what you send
- Simpler architecture
- Less mapping code

### When SQLModel Is Appropriate

**✅ Good use cases:**

- Database structure mirrors API contract
- Simple CRUD applications
- API closely matches database structure
- You want minimal boilerplate
- Team prefers single source of truth

**❌ Bad use cases:**

- Complex business logic transformations needed
- API response differs significantly from storage
- Need to hide internal fields from API
- Security requires field filtering
- Multiple representation types needed

**Real-world decision:** SQLModel works best for ~60-70% of endpoints. Complex endpoints still benefit from separate schemas.

---

## Part 3: Architectural Implications

### Concept: When to Use SQLModel vs Pydantic+SQLAlchemy

**Use SQLModel when:**

```
User database structure = User API response structure
No sensitive fields to hide
Simple validation matches database constraints
Quick prototyping preferred
```

**Use Pydantic+SQLAlchemy when:**

```
Database has implementation details API shouldn't expose
Complex transformation logic needed
Different representations for different users
Security filtering required
```

### Design Pattern: Layered Approach

Smart teams use BOTH in same app:

```
Simple endpoints:     SQLModel (less code)
Complex endpoints:    Separate schemas (more control)

Example:
GET /users/{id}       → Use SQLModel (simple)
POST /users/search    → Separate schemas (complex filtering)
PUT /admin/users/{id} → Separate schemas (permission checks)
```

### The Composition Over Inheritance Principle

**Concept: Building blocks instead of inheritance**

With SQLModel, resist urge to create base models and inherit. Instead, think of fields as composable:

```
WRONG: Create UserBase, inherit in 5 places
RIGHT: Define field constraints once, reuse the model

Philosophy shift:
SQLAlchemy: Think in terms of inheritance hierarchies
SQLModel: Think in terms of "Is this model appropriate here?"
```

---

# QUERY OPTIMIZATION & N+1 PROBLEMS - 3 HOURS

## Part 1: Understanding N+1 Query Problem

### Formal Definition: N+1 Query Problem

**N+1 Query Problem:** When fetching N parent objects requires:

- 1 query to fetch N parents
- N additional queries (one per parent) to fetch children
- Total: 1 + N queries instead of 1-2 queries

### Why It Happens

**Root cause concept:** Lazy loading

When you define a relationship in ORM:

```
class User:
    posts = relationship("Post")  # Lazy-loaded by default
```

The ORM waits until you access `.posts` to query for it:

```
users = db.query(User).all()  # Query 1: Gets 100 users

for user in users:
    print(user.posts)  # Query 2-101: Gets posts for each user!
    # Total: 101 queries!
```

### Real-World Impact

**Performance scenario:**

```
Without N+1 optimization:
- Fetch 100 users: 1 query
- Access each user's posts: 100 queries
- Total: 101 queries
- Time: 5 seconds (with 50ms per query)

With optimization:
- Fetch 100 users with posts: 2 queries (join)
- Time: 100ms

Performance difference: 50x faster!
```

---

## Part 2: N+1 Solutions and When to Use Them

### Solution 1: Eager Loading (Concept)

**What it is:** Load related data immediately with parent, not on-demand

**When to use:**

- Related data always needed with parent
- Relationship has small number of items
- Performance matters more than memory

**How it works conceptually:**

```
Instead of:
1. Load user
2. Wait for access to posts
3. Load posts

Do this:
1. Load user and posts together
2. Everything ready immediately
```

### Solution 2: Explicit Joins (Concept)

**What it is:** Query the join yourself, controlling exactly what's loaded

**When to use:**

- Need to filter by related data
- Want to load multiple relationships selectively
- Performance critical, need control
- Different views need different related data

**Conceptual advantage:** You know exactly what query runs. No surprises.

### Solution 3: Batch Loading (Concept)

**What it is:** Load IDs first, then all related objects in one query

**Pattern:**

```
1. Load parents (get their IDs)
2. Load all children where parent_id IN (ids)
3. Associate in memory

Queries: 2 instead of N+1
```

**When to use:**

- Can't join directly
- Related data might not exist for all parents
- Want flexibility in what gets loaded

### Solution 4: GraphQL Approach (Concept)

**What it is:** Let client specify exactly what data they need

**Concept:** Server loads only what client requests

```
Client says: "Give me users with posts"
Server loads: Users + Posts

Client says: "Give me users"  
Server loads: Users only (no posts)
```

**When to use:**

- Multiple clients with different needs
- Complex data relationships
- Want to minimize bandwidth and queries

---

## Part 3: Query Patterns and Thinking Models

### Concept: Thinking in Sets, Not Loops

**Traditional loop thinking (creates N+1):**

```
Get users
For each user:
    Get user's data
    Get user's posts
    Get user's comments
    
Result: 1 + 3N queries
```

**Set thinking (optimizes queries):**

```
Get all users with posts and comments (1-2 joins)
Process all together

Result: 1-2 queries total
```

### Concept: The Cost of Flexibility

**Trade-off principle:**

```
More flexible loading (lazy) 
→ Easier to code initially
→ Hidden N+1 problems
→ Suddenly slow

Less flexible loading (eager)
→ Must plan upfront
→ Explicit queries
→ Predictable performance
```

### Concept: Query Optimization Mentality

**Shift in thinking required:**

```
Week 1-5 mentality:
"How do I write this query?"

Week 6 mentality:
"How many queries will this create?"
"How many times will the database be hit?"
"Can I batch this differently?"
"Is this query actually happening?"
```

---

# PERFORMANCE FUNDAMENTALS - 2.5 HOURS

## Part 1: Database Connection Pooling

### Formal Definition: Connection Pool

**Connection Pool:** A cache of database connections that can be reused rather than creating new ones for each request.

### Why Pooling Matters (The Concept)

**Without pooling:**

```
Every request:
1. Create TCP connection to database
2. Authenticate  
3. Execute query
4. Close connection

Time overhead: 50-200ms per request just for connection!
```

**With pooling:**

```
Every request:
1. Get existing connection from pool
2. Execute query
3. Return connection to pool

Time overhead: <1ms per request
```

### Conceptual Architecture

```
Connection Pool (maintains 5-20 connections):
┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐
│  Conn   │ │  Conn   │ │  Conn   │ │  Conn   │ │  Conn   │
│ idle    │ │ active  │ │ idle    │ │ active  │ │ idle    │
└─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘

Request 1: Takes idle connection → executes → returns to pool
Request 2: Takes idle connection → executes → returns to pool
Request 100: Waits for connection if all busy, or gets from pool
```

### Key Concepts

**Pool size balance:**

```
Too small (size=2):
- Requests queue up
- Slow responses
- Database underutilized

Too large (size=100):  
- Connections unused
- Memory waste
- Database resource strain

Optimal (size=5-20):
- Most requests get immediate connection
- Reasonable memory usage
- Database resources efficient
```

**Pool pre-ping concept:**

```
Problem: Connection sits in pool, goes stale
Server closes idle connection after 30 minutes
Request uses stale connection
Error!

Solution: Before using pooled connection, ping it
If dead: reconnect
If alive: use
```

---

## Part 2: Database Indexing Principles

### Formal Definition: Index

**Database Index:** A data structure that speeds up data retrieval by creating a sorted lookup table.

### Conceptual Model of Indexes

**Without index (full table scan):**

```
Query: Find user with username='alice'

Database does:
1. Open users table
2. Read row 1: Check if 'alice'? No
3. Read row 2: Check if 'alice'? No
4. Read row 3: ... 
5. ... (scan all 1 million rows)
6. Read row 1,000,000: Check if 'alice'? Yes!

Time: O(n) - proportional to table size
```

**With index (tree search):**

```
Index on username:
    ╔═══════╗
    │  bob  │
   ╱         ╲
 ╔════════╗  ╔════════╗
 │ alice  │  │ charlie│
 
Query: Find user with username='alice'

Database does:
1. Check root: 'bob' - alice < bob? Yes, go left
2. Check left: 'alice' - match!
3. Get row

Time: O(log n) - logarithmic
```

### When to Index (The Concept)

**Index speeds up:**

- WHERE conditions: `WHERE username = 'alice'` ✓
- Sorting: `ORDER BY created_at` ✓
- Joins: `WHERE post.user_id = 1` ✓

**Index doesn't help:**

- Rare queries
- Queries on non-selective data
- Writing (indexes slow down INSERT/UPDATE)

### Index Trade-offs (Strategic Thinking)

```
Index benefits:
- Faster reads
- Better query performance

Index costs:
- Slower writes (must update index)
- Memory usage (index is stored)
- Maintenance overhead

Decision: Index high-read, low-write columns
Don't index everything
```

---

## Part 3: Query Planning and Analysis

### Concept: EXPLAIN and Query Plans

**Query Plan:** Database's step-by-step execution strategy for a query.

### Understanding Query Plans (Conceptually)

**Good plan:**

```
Uses index → Fast lookup → Return results
Complexity: O(log n)
```

**Bad plan:**

```
Full table scan → Check every row → Return results  
Complexity: O(n)
```

**Knowledge principle:** If you don't know your query plan, you don't know if it's fast.

### Real-world Decision Making

**Concept: Measure Before Optimizing**

```
Rule: You can't optimize what you don't measure

Process:
1. Know which queries are slow (monitoring)
2. Know the query plan (EXPLAIN)
3. Know if index helps (test with/without)
4. Know if change is worth it (cost-benefit)
```

Don't optimize blindly. Data-driven optimization.

---

# CACHING PATTERNS & STRATEGIES - 2.5 HOURS

## Part 1: Caching Fundamentals

### Formal Definition: Cache

**Cache:** A fast but limited storage layer that holds copies of frequently accessed data to reduce need to fetch from slower source.

### Why Caching Matters

**The speed hierarchy:**

```
L1: CPU cache (nanoseconds)
L2: Memory (microseconds)  
L3: SSD (milliseconds)
L4: Database network (10s-100s milliseconds)
L5: Remote API (seconds)

Caching moves data to higher tiers
Eliminates expensive trips to lower tiers
```

### Caching Trade-off (Core Concept)

```
No cache:
- Always up-to-date
- Slower responses
- More database load
- Simple logic

With cache:
- Potentially stale
- Faster responses  
- Less database load
- Complex invalidation logic
```

Every caching decision is trading staleness for speed.

---

## Part 2: Caching Strategies

### Strategy 1: Time-Based Cache (TTL)

**Concept:** Cache expires after fixed time

```
Cache expires after 60 seconds

Age 0 seconds:  Fresh from cache
Age 30 seconds: Still in cache
Age 60 seconds: Expires, fetch fresh
Age 61 seconds: Fresh from cache (new set)
```

**When to use:**

- Data changes predictably
- Staleness acceptable
- Simple to implement
- No invalidation needed

**Example:** Stock prices (update every minute), weather (update hourly)

### Strategy 2: Dependency-Based Cache

**Concept:** Cache expires when underlying data changes

```
Cache post details
User updates post
Cache invalidates
Next request: Fresh data

Dependencies: When to invalidate
- User edited post → invalidate post cache
- Post deleted → invalidate post + user's posts cache
- Comment added → invalidate post's comments cache
```

**When to use:**

- Data changes unpredictably
- Staleness unacceptable
- Know what changes matter
- More complex logic acceptable

**Challenge:** Tracking all dependencies gets complex quickly

### Strategy 3: Cache-Aside (Lazy Loading)

**Concept:** Load from cache if available, otherwise fetch and cache

```
Request for user 1:
1. Check cache: miss
2. Query database
3. Store in cache
4. Return data

Next request for user 1:
1. Check cache: hit
2. Return cached data

Invalidation: Manual or TTL
```

**When to use:**

- Variable access patterns
- Don't know what to cache upfront
- Cost of cache miss acceptable

### Strategy 4: Write-Through Cache

**Concept:** Always write to cache AND database simultaneously

```
Write request:
1. Update cache
2. Update database
3. Return success

Goal: Cache always in sync with database
```

**When to use:**

- Data consistency critical
- Can tolerate extra write latency
- Relatively few writes

### Strategy 5: Write-Behind Cache (Write-Back)

**Concept:** Write to cache immediately, database later

```
Write request:
1. Update cache
2. Queue database update
3. Return success immediately

Later: Batch write to database
```

**When to use:**

- Can tolerate temporary inconsistency
- High write volume
- Need response speed

**Risk:** If system crashes before database write, data lost

---

## Part 3: Caching Patterns in FastAPI

### Pattern 1: Simple Function Caching

**Concept:** Cache function result based on arguments

```
def get_expensive_data(user_id):
    # If called twice with same user_id
    # Should return cached result second time
    
    # Pattern: Check cache → Hit? Return : Fetch and cache
```

### Pattern 2: Cache Layers

**Concept:** Multiple cache levels for different data

```
Layer 1 (fastest): In-memory cache (Python dict)
- Holds hot data
- Per-server
- Lost on restart

Layer 2 (medium): Redis cache
- Across all servers
- Persistent restart
- Network latency

Layer 3 (slowest): Database
- Source of truth
- Always available
- Slow
```

### Pattern 3: Cache Busting Strategy

**Concept:** Methods to ensure cache gets refreshed when needed

```
Strategies:
1. Time-based: Expire after N seconds
2. Event-based: Invalidate on specific events
3. Dependency-based: Track what data depends on what
4. Manual: Admin endpoint to clear cache
5. Lazy: Next request after deletion fetches fresh
```

### Key Caching Insight

**Concept: Cache is optimization, not architecture**

```
Wrong approach:
- Build cache into core architecture
- Everything goes through cache
- Cache failures break system

Right approach:
- Cache is optional layer
- Works with or without cache
- Cache failure degrades gracefully (slower, not broken)
```

---

# TESTING WITH DATABASES - 2 HOURS

## Part 1: Testing Challenges

### Challenge 1: Isolation

**Problem:** Tests interfere with each other

```
Test 1 creates user 'alice'
Test 2 creates user 'alice'  
Test 1 expects count=1
Test 2 expects count=1
Test 3 runs in parallel, creates 'alice'
All three fail due to uniqueness constraint!
```

**Concept:** Each test needs clean database state

### Challenge 2: Speed

**Problem:** Real database operations are slow

```
Test 1: Create user, create post, test → 500ms
Test 2: Create user, create post, test → 500ms
Test 100: ... → 50 seconds for 100 simple tests!

Running test suite now takes too long
Developers stop running tests
```

**Concept:** Tests must be fast to be useful

### Challenge 3: Environment

**Problem:** Test database might behave differently than production

```
SQLite (testing): No foreign key constraints by default
PostgreSQL (production): Enforces constraints

Test passes with SQLite
Production fails with PostgreSQL
```

**Concept:** Test environment should mirror production

---

## Part 2: Testing Strategies

### Strategy 1: Real Database (SQLite)

**Approach:** Use real database, create/destroy for each test

**Advantages:**

- Tests real database behavior
- No mocks or stubs
- Finds integration issues

**Disadvantages:**

- Slow (setup/teardown overhead)
- Disk I/O for every test
- Not isolated (tests can interfere)

**When to use:** Integration tests, not unit tests

### Strategy 2: In-Memory Database

**Approach:** Use SQLite in-memory (`:memory:`) for tests

**Advantages:**

- Fast (RAM, no disk)
- Clean for each test
- Mimics real database

**Disadvantages:**

- Doesn't test persistence
- In-memory quirks differ from disk
- Limited to SQLite

**When to use:** Most unit/integration tests

### Strategy 3: Fixtures and Factories

**Concept:** Pre-built test data patterns

```
Fixture: Known, reusable test state
- "Populated database with 10 users"
- "User with admin role"
- "Post with comments"

Factory: Generator for test objects
- Can create variations
- Reduces duplication
- Clearer test intent
```

**When to use:** Every test that needs data

### Strategy 4: Transaction Rollback

**Concept:** Run test inside transaction, roll back after

```
Test:
1. Start transaction
2. Create test data
3. Run test
4. Check results
5. Rollback (undo all changes)
6. Database clean for next test

Advantage: Real database, zero cleanup time
```

---

## Part 3: Testing Philosophy

### Principle: Testing Pyramid

**Concept:** Different types of tests in proportion

```
       ▲
       │ End-to-end (few, slow, complete)
       │
      ╱ ╲ Integration (medium, moderate speed)
     ╱   ╲
    ╱     ╲ Unit tests (many, fast)
   ╱───────╲
```

**Application to database testing:**

```
Unit tests (many): Mock database, test logic
Integration tests (some): Real database, test queries
E2E tests (few): Real system, test workflows
```

### Principle: Test Characteristics

**Good database test:**

- ✓ Isolated (doesn't affect other tests)
- ✓ Repeatable (same result every time)
- ✓ Fast (milliseconds, not seconds)
- ✓ Clear (test intent obvious)
- ✓ Maintainable (easy to update)

**Bad database test:**

- ✗ Depends on other tests
- ✗ Flaky (fails randomly)
- ✗ Slow (seconds per test)
- ✗ Unclear (need to read code to understand)
- ✗ Brittle (breaks on minor changes)

---

# WEEK 6 CHECKPOINT PROJECT

## Project: Blog API - Production Ready

### Objective

Take your Week 5 Blog API and make it production-ready by:

1. Deciding when to use SQLModel vs separate schemas
2. Identifying and fixing N+1 queries
3. Adding strategic indexes
4. Implementing caching layer
5. Writing comprehensive tests

### Part 1: SQLModel Decision Points

**Task:** For each endpoint, decide:

- Use SQLModel (simple case)?
- Use Pydantic + SQLAlchemy (complex case)?
- Justify your decision

**Endpoints to analyze:**

- GET /users/ - List users
- POST /users/ - Create user
- GET /posts/{id} - Get single post
- GET /users/{id}/posts - Get user's posts

**Decision criteria:**

- Does API response match database schema exactly?
- Are there fields to hide?
- Is transformation needed?

### Part 2: N+1 Query Analysis

**Task:** Identify N+1 problems in your API

**Analysis steps:**

1. Add query logging (see which queries execute)
2. Test endpoints with multiple records
3. Count queries executed
4. Identify where N+1 happens

**Examples to find:**

- Getting users then accessing `.posts` on each
- Getting posts then accessing `.author` on each
- Nested relationship access

### Part 3: Query Optimization

**Task:** Fix identified N+1 problems

**Methods available:**

- Eager loading
- Explicit joins
- Batch loading
- GraphQL-like approach

Choose method based on:

- Access patterns
- Relationship sizes
- Performance needs

### Part 4: Strategic Indexing

**Task:** Add indexes where they matter

**Analysis:**

- Which columns are in WHERE clauses?
- Which columns are used for sorting?
- Which columns are foreign keys?

**Implementation:**

- Create indexes
- Test query plans
- Measure improvement

### Part 5: Caching Implementation

**Task:** Add caching to expensive operations

**Strategy selection:**

- User data: TTL-based (5 minute cache)
- Post data: Dependency-based (invalidate on user edit)
- List endpoints: TTL-based (1 minute)

**Tools:**

- Python lru_cache (simple)
- Redis (distributed)
- Dependency tracking (custom)

### Part 6: Test Suite

**Task:** Write tests demonstrating understanding

**Test categories:**

- Unit tests (mock database)
- Integration tests (real database)
- Performance tests (query counts)

**What to test:**

- CRUD operations work
- Relationships load correctly
- N+1 optimizations effective
- Cache invalidation works
- Indexes improve performance

### Deliverables

1. **Architecture Decision Document**
    
    - Which endpoints use SQLModel?
    - Which use separate schemas?
    - Rationale for each
2. **Performance Analysis Report**
    
    - N+1 problems identified
    - Query plans analyzed
    - Optimization strategy
    - Benchmark results
3. **Implementation Changes**
    
    - Code using optimization strategies
    - Caching layer
    - Indexes
    - Tests

### Evaluation Checklist

**Conceptual Understanding:**

- [ ] Can explain when to use SQLModel vs schemas
- [ ] Can identify N+1 problems from query logs
- [ ] Can explain connection pooling benefits
- [ ] Can discuss index trade-offs
- [ ] Can choose appropriate caching strategy

**Implementation:**

- [ ] Query optimization measurably improves performance
- [ ] Caching reduces database load
- [ ] Tests verify optimizations work
- [ ] Code is maintainable and documented

---

# KEY CONCEPTS SUMMARY

## SQLModel

- Combines Pydantic + SQLAlchemy
- Single source of truth
- Appropriate for 60-70% of endpoints
- Trade-off: Simplicity vs flexibility

## Query Optimization

- N+1 problem: 1 + N queries instead of 1-2
- Solutions: Eager loading, joins, batch loading
- Mentality shift: Think in sets, not loops
- Measurement critical before optimizing

## Performance Principles

- Connection pooling: Cache connections (50-100x speedup)
- Indexing: Trade write cost for read speed
- Query plans: Know how database executes
- Measurements: Optimize what you measure

## Caching Strategies

- Time-based (TTL): Simple, eventual consistency
- Dependency-based: Complex but accurate
- Cache-aside: Lazy loading pattern
- Write-through/Write-behind: Trade consistency for speed

## Testing Philosophy

- Isolation: Each test independent
- Speed: Fast enough to run often
- Environment: Mirrors production
- Pyramid: Few E2E, some integration, many units

---

# THINKING FRAMEWORKS

## Decision Framework: Cache or Not?

```
Does data change frequently?
├─ Yes → Don't cache (staleness unacceptable)
└─ No → Adequate for caching

Is it accessed frequently?
├─ Yes → Cache provides value
└─ No → Caching overhead not worth it

Can we tolerate staleness?
├─ Yes → Use TTL-based
└─ No → Use dependency-based or none

Result: Cache if infrequent changes + frequent access
```

## Decision Framework: SQLModel vs Schemas

```
Does API response = DB structure exactly?
├─ Yes ─┐
│       └─ Use SQLModel
└─ No  ─┬─ Are transformations simple?
        ├─ Yes → Use SQLModel with adjustments
        └─ No → Use separate schemas
```

## Performance Debugging Checklist

```
System slow?
├─ Check connection pooling
│  └─ Pool size adequate?
├─ Check query volume
│  └─ N+1 problems present?
├─ Check query plans
│  └─ Using indexes correctly?
├─ Check cache hit rates
│  └─ Cache working?
└─ Check bottleneck location
   └─ Database? Network? Code?
```

---

# NEXT STEPS AFTER WEEK 6

You've now completed:

- ✅ Python foundations (Week 1)
- ✅ FastAPI basics (Week 2)
- ✅ Advanced routing & validation (Week 3)
- ✅ Dependencies & middleware (Week 4)
- ✅ Databases (Week 5)
- ✅ Advanced patterns & optimization (Week 6)

**Next logical steps:**

- Week 7: Authentication & Authorization (JWT, OAuth2)
- Week 8: Real-time Features (WebSockets)
- Week 9: Background Tasks & Async
- Week 10: Deployment & DevOps
- Week 11: Monitoring & Observability
- Week 12: Advanced FastAPI Patterns

You're ready to build production-grade APIs. Choose your next focus based on application needs.

---

# RESOURCES FOR DEEPER LEARNING

**Documentation:**

- SQLAlchemy Query Guide (read the query patterns)
- Redis Documentation (if choosing Redis for caching)
- PostgreSQL Query Planning (for index understanding)

**Conceptual Reading:**

- "High Performance MySQL" (database concepts transfer)
- "Release It!" (performance and operational thinking)
- "Designing Data-Intensive Applications" (distributed systems thinking)

**Tools for Learning:**

- `EXPLAIN ANALYZE` in your database
- Query logging in SQLAlchemy
- Performance monitoring with timing decorators
- Load testing tools (locust, wrk)

---

**End of Week 6 Content**

Congratulations! You've progressed from basic API building to production-grade architecture thinking. The focus has shifted from "how do I make this work" to "how do I make this work efficiently and maintainably."

This is where junior developers become senior developers - in understanding trade-offs and making informed architectural decisions. 🎓

**Next week:** Ready to add authentication, or would you prefer to dive deeper into any Week 6 concepts?