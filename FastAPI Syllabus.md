# FastAPI Mastery Syllabus (10-12 Weeks)

**Your Profile:** Intermediate Python | API Basics Known | 20-30 hrs/week | Balanced Learning

**Goal:** Production-ready FastAPI expertise with deep understanding

---

## 📊 Timeline Overview

|Phase|Duration|Hours/Week|Focus|
|---|---|---|---|
|**Phase 1: Foundations**|Weeks 1-2|20-25 hrs|Decorators, Async/Await, FastAPI Basics|
|**Phase 2: Core API Development**|Weeks 3-4|25-28 hrs|Routing, Request/Response, Validation|
|**Phase 3: Database & Data**|Weeks 5-6|25-28 hrs|SQLAlchemy, SQLModel, Database Patterns|
|**Phase 4: Authentication & Security**|Weeks 7-8|25-28 hrs|JWT, OAuth2, Security Best Practices|
|**Phase 5: Advanced Features**|Weeks 9-10|25-28 hrs|WebSockets, Background Tasks, Streaming|
|**Phase 6: Production & Deployment**|Weeks 11-12|20-25 hrs|Testing, Monitoring, Docker, Deployment|

**Total Time Investment:** 140-165 hours

---

## 🚀 PHASE 1: FOUNDATIONS (Weeks 1-2)

### Week 1: Python Prerequisites (12-13 hrs)

**Topics:**

- **Decorators (3 hrs theory + practice)**
    
    - Function decorators basics
    - Decorators with arguments
    - Stacking decorators
    - Practical decorator patterns used in FastAPI
- **Async/Await & Concurrency (4 hrs theory + practice)**
    
    - Event loops and asyncio
    - async def vs def
    - await keyword
    - Concurrent task execution
    - Why FastAPI is async-first
- **Type Hints Deep Dive (3 hrs)**
    
    - Generic types (List[str], Dict[str, int])
    - Optional and Union types
    - Type hints for functions
    - Why Pydantic needs type hints
- **Built-in Tools (2 hrs)**
    
    - Context managers (with statement)
    - Generators and yield
    - Comprehensions

**Time Breakdown:**

- Theory: 6 hrs (videos + documentation)
- Hands-on Coding: 6-7 hrs (exercises)

**Checkpoint Project:** Build 3 custom decorators (timer, retry, validator)

---

### Week 2: FastAPI Fundamentals (12-13 hrs)

**Topics:**

- **What is FastAPI? (1.5 hrs)**
    
    - FastAPI vs Flask vs Django
    - ASGI vs WSGI
    - Architecture overview
    - Why Starlette and Pydantic matter
- **Your First API (3 hrs theory + practice)**
    
    - Project setup (virtual env, pip install fastapi uvicorn)
    - Creating first endpoint
    - Running the server
    - Interactive docs (Swagger UI, ReDoc)
    - Path operations and HTTP methods
- **Request & Response Basics (4 hrs)**
    
    - Path parameters
    - Query parameters
    - Request body
    - Response models
    - Status codes
    - Headers
- **Pydantic Models (3 hrs)**
    
    - Creating base models
    - Field validation
    - Nested models
    - Model configuration
- **Dependencies Intro (1.5 hrs)**
    
    - What are dependencies
    - Dependency injection pattern
    - Simple dependency example

**Time Breakdown:**

- Theory/Reading: 5 hrs
- Coding Projects: 7-8 hrs

**Checkpoint Project:** Build a simple Todo API with GET, POST endpoints (no database yet)

---

## 📝 PHASE 2: CORE API DEVELOPMENT (Weeks 3-4)

### Week 3: Advanced Routing & Validation (13 hrs)

**Topics:**

- **Path Operations Deep Dive (2.5 hrs)**
    
    - Multiple path parameters
    - Numeric validations (gt, gte, lt, lte)
    - String validations (regex, min_length, max_length)
    - Enum paths
    - Documentation strings and examples
- **Request Details (3 hrs)**
    
    - Body, path, query, header, cookie parameters
    - Multiple body parameters
    - Singular values in body
    - Form data
    - File uploads (single and multiple)
- **Response Details (2.5 hrs)**
    
    - Response models
    - Response status codes
    - Response headers
    - Custom responses (JSONResponse, FileResponse, etc.)
    - Docstring examples for auto-docs
- **Error Handling (2.5 hrs)**
    
    - HTTPException
    - Custom exception handlers
    - Request validation errors
    - Error response models
- **Documentation (1.5 hrs)**
    
    - OpenAPI schema
    - How to write good endpoint documentation
    - Example values and schemas

**Time Breakdown:**

- Theory: 6 hrs
- Coding: 7 hrs

**Checkpoint Project:** Build a User Management API with validation (create, read, update, delete users with proper validation)

---

### Week 4: Dependencies & Middleware (12-13 hrs)

**Topics:**

- **Dependencies System (4.5 hrs)**
    
    - Dependency functions
    - Classes as dependencies
    - Sub-dependencies
    - Dependency caching
    - Database session injection pattern
    - Shared resources
- **Middleware Concepts (3 hrs)**
    
    - What is middleware
    - Adding middleware to FastAPI
    - Common middleware patterns
    - CORS configuration
    - Custom middleware
- **CORS, HTTPS, Others (2 hrs)**
    
    - CORS headers and configuration
    - HTTPS and security headers
    - Request/response lifecycle
- **Advanced Error Handling (2 hrs)**
    
    - Creating reusable error handlers
    - Global exception handling
    - Validation error customization
- **Testing Basics (1.5 hrs)**
    
    - TestClient from fastapi.testclient
    - Writing basic tests
    - Test structure

**Time Breakdown:**

- Theory: 6 hrs
- Coding: 6-7 hrs

**Checkpoint Project:** Refactor Todo API with dependency injection for authentication and logging. Add comprehensive tests.

**Phase 1-2 Milestone:** You should be able to build a fully functional API without database.

---

## 💾 PHASE 3: DATABASE & DATA (Weeks 5-6)

### Week 5: Database Fundamentals & SQLAlchemy (13 hrs)

**Topics:**

- **SQL & Database Basics (2 hrs)**
    
    - Relational databases overview
    - SQL basics (SELECT, INSERT, UPDATE, DELETE)
    - Relationships (One-to-Many, Many-to-Many)
    - Database design principles
- **SQLAlchemy ORM (5 hrs)**
    
    - Installation and setup
    - Declarative base and table definition
    - Sessions and transactions
    - CRUD operations
    - Relationships (relationships, foreign keys)
    - Query building and filtering
    - Lazy loading vs eager loading
- **Alembic Migrations (2.5 hrs)**
    
    - What are migrations
    - Creating migrations
    - Running migrations
    - Rollback strategies
    - Best practices
- **SQLAlchemy + FastAPI Integration (2.5 hrs)**
    
    - Database setup in FastAPI
    - Session dependency
    - Creating models
    - Using models in endpoints
- **Connection Pooling & Performance (1 hr)**
    
    - Connection pools
    - Query optimization basics

**Time Breakdown:**

- Theory: 7 hrs
- Hands-on: 6 hrs

**Checkpoint Project:** Migrate Todo API to use SQLite with SQLAlchemy. Implement full CRUD operations.

---

### Week 6: SQLModel & Advanced Database Patterns (12-13 hrs)

**Topics:**

- **SQLModel (3.5 hrs)**
    
    - What is SQLModel (combines Pydantic + SQLAlchemy)
    - Creating models with SQLModel
    - Benefits over separate Pydantic/SQLAlchemy models
    - Relationships in SQLModel
    - Querying with SQLModel
- **Advanced Relationships (3 hrs)**
    
    - One-to-Many implementation
    - Many-to-Many implementation
    - Cascade options
    - Relationship loading strategies
    - Circular relationships handling
- **Database Best Practices (2 hrs)**
    
    - Indexing strategies
    - Query optimization
    - N+1 query prevention
    - Database transactions
- **Complex Queries (2 hrs)**
    
    - Filtering and sorting
    - Pagination
    - Search implementations
    - Aggregations (count, sum, etc.)
- **Error Handling with Databases (1.5 hrs)**
    
    - Handling database errors
    - Integrity constraint errors
    - Connection errors
- **Testing with Databases (1.5 hrs)**
    
    - In-memory testing databases
    - Test fixtures
    - Database state management in tests

**Time Breakdown:**

- Theory: 6.5 hrs
- Coding: 5.5-6.5 hrs

**Checkpoint Project:** Build a Blog API with Posts, Authors, Comments, and Categories (Many-to-Many relationships). Full CRUD with complex queries.

**Phase 3 Milestone:** Production-ready database interactions with proper relationships and migrations.

---

## 🔐 PHASE 4: AUTHENTICATION & SECURITY (Weeks 7-8)

### Week 7: JWT & OAuth2 (13 hrs)

**Topics:**

- **Security Concepts (1.5 hrs)**
    
    - Password hashing (bcrypt, argon2)
    - Tokens vs Sessions
    - Stateless authentication
    - Security best practices
- **JWT (JSON Web Tokens) (3.5 hrs)**
    
    - JWT structure (header, payload, signature)
    - Creating JWT tokens
    - Validating JWT tokens
    - Token expiration and refresh
    - PyJWT library
- **OAuth2 & Scopes (3.5 hrs)**
    
    - OAuth2 flow
    - FastAPI OAuth2PasswordBearer
    - Token creation for OAuth2
    - Scopes and permissions
    - Token validation
- **User Registration & Login (2.5 hrs)**
    
    - User creation endpoint
    - Login endpoint with token generation
    - Password hashing best practices
    - Token response format
- **Protecting Endpoints (1.5 hrs)**
    
    - Current user dependency
    - Token validation middleware
    - Scope-based access control
- **Introduction to HTTPS (1 hr)**
    
    - Why HTTPS matters
    - Certificate basics

**Time Breakdown:**

- Theory: 7 hrs
- Coding: 6 hrs

**Checkpoint Project:** Add JWT authentication to Blog API. Implement user registration, login, and protect endpoints based on user ownership.

---

### Week 8: Advanced Security & Authorization (12-13 hrs)

**Topics:**

- **Permission & Authorization Systems (3 hrs)**
    
    - Role-based access control (RBAC)
    - Permission-based systems
    - Implementing custom authorization
    - Admin vs user roles
- **Advanced OAuth2 Scenarios (2.5 hrs)**
    
    - Refresh tokens
    - Revocation strategies
    - Multiple authentication methods
    - Social login setup overview
- **Security Headers & CORS Deep Dive (2 hrs)**
    
    - CORS configuration for production
    - Security headers (CSP, X-Frame-Options, etc.)
    - HTTPS enforcement
- **Rate Limiting & Throttling (1.5 hrs)**
    
    - Why rate limiting matters
    - Implementing rate limiting
    - User-based vs IP-based limiting
- **Secrets Management (1.5 hrs)**
    
    - Environment variables
    - .env files and python-dotenv
    - Secret rotation
    - Storing sensitive data
- **Comprehensive Security Testing (1.5 hrs)**
    
    - Testing auth endpoints
    - Testing permission checks
    - Security-focused test cases
- **Logging & Monitoring Security (1 hr)**
    
    - Logging authentication events
    - Detecting suspicious activity

**Time Breakdown:**

- Theory: 6 hrs
- Coding: 6-7 hrs

**Checkpoint Project:** Enhance Blog API with role-based access (Admin, Author, Reader). Implement rate limiting on endpoints. Comprehensive security tests.

**Phase 4 Milestone:** Enterprise-grade authentication and authorization system.

---

## ⚡ PHASE 5: ADVANCED FEATURES (Weeks 9-10)

### Week 9: WebSockets & Real-time Features (13 hrs)

**Topics:**

- **WebSocket Fundamentals (2.5 hrs)**
    
    - WebSocket vs HTTP
    - Connection lifecycle
    - Bidirectional communication
    - When to use WebSockets
- **FastAPI WebSocket Implementation (4 hrs)**
    
    - Creating WebSocket endpoints
    - Accepting connections
    - Sending/receiving messages
    - Handling disconnections
    - Multiple concurrent connections
- **Real-time Use Cases (2 hrs)**
    
    - Chat systems
    - Live notifications
    - Real-time data updates
    - Collaborative features
- **WebSocket with Dependencies (1.5 hrs)**
    
    - Authenticating WebSocket connections
    - Using dependencies in WebSockets
    - Session management
- **Broadcasting & Pub/Sub (2 hrs)**
    
    - Broadcasting to multiple clients
    - Pub/Sub patterns
    - Message routing
- **Testing WebSockets (1 hr)**
    
    - WebSocket test client
    - Testing connection lifecycle
    - Message verification

**Time Breakdown:**

- Theory: 6.5 hrs
- Coding: 6.5 hrs

**Checkpoint Project:** Add real-time chat/notifications to Blog API. Users can send messages that appear instantly for others.

---

### Week 10: Background Tasks, Streaming & Advanced Patterns (12-13 hrs)

**Topics:**

- **Background Tasks (3 hrs)**
    
    - BackgroundTasks from FastAPI
    - Task queuing basics
    - Celery overview and setup
    - Task scheduling with APScheduler
    - Long-running operations
- **File Streaming & Uploads (3 hrs)**
    
    - File upload handling
    - Large file uploads
    - Streaming responses
    - Streaming file downloads
    - File validation
- **Server-Sent Events (SSE) (1.5 hrs)**
    
    - SSE vs WebSockets
    - Implementing SSE
    - Use cases
- **Caching Strategies (2 hrs)**
    
    - Response caching
    - Redis integration
    - Cache invalidation
    - Caching best practices
- **Event Hooks & Lifespan (1.5 hrs)**
    
    - Application startup/shutdown
    - Resource initialization
    - Cleanup operations
- **Advanced Testing Patterns (1.5 hrs)**
    
    - Testing async code
    - Mocking external services
    - Testing background tasks

**Time Breakdown:**

- Theory: 6.5 hrs
- Coding: 5.5-6.5 hrs

**Checkpoint Project:** Add email notifications (using Celery), file uploads, and SSE for real-time activity feed to Blog API.

**Phase 5 Milestone:** Modern, scalable features with real-time capabilities.

---

## 🚢 PHASE 6: PRODUCTION & DEPLOYMENT (Weeks 11-12)

### Week 11: Testing, Monitoring & Performance (13 hrs)

**Topics:**

- **Comprehensive Testing (4 hrs)**
    
    - Unit tests vs integration tests
    - pytest best practices
    - Test fixtures and factories
    - Mocking and patching
    - Code coverage analysis
    - Test organization and structure
- **Logging & Monitoring (3 hrs)**
    
    - Application logging setup
    - Structured logging (JSON logs)
    - Log levels and rotation
    - Monitoring tools (ELK, Datadog, etc.)
    - Health check endpoints
- **Performance Optimization (3 hrs)**
    
    - Profiling tools (cProfile, py-spy)
    - Query optimization
    - Caching strategies
    - Async performance benefits
    - Load testing tools (locust, wrk)
- **Documentation (2 hrs)**
    
    - Auto-generated documentation
    - Additional markdown documentation
    - README and setup guides
    - API change logs
- **API Versioning (1 hr)**
    
    - Versioning strategies
    - Backward compatibility
    - Deprecation handling

**Time Breakdown:**

- Theory: 6.5 hrs
- Coding & Practical: 6.5 hrs

**Checkpoint Project:** Add comprehensive test suite (80%+ coverage), implement logging, run load tests, generate documentation.

---

### Week 12: Deployment & Advanced Operations (12-13 hrs)

**Topics:**

- **Containerization with Docker (3.5 hrs)**
    
    - Dockerfile creation
    - Multi-stage builds
    - Docker Compose for local development
    - Environment configuration
    - Docker best practices
- **Deployment Platforms (3 hrs)**
    
    - Heroku (easiest)
    - AWS (EC2, ECS, Lambda)
    - DigitalOcean
    - Railway, Render (modern alternatives)
    - Environment-specific configurations
- **Production Setup (2.5 hrs)**
    
    - Gunicorn/Uvicorn configuration
    - Reverse proxy (Nginx)
    - SSL/TLS certificates
    - Load balancing
- **Database in Production (1.5 hrs)**
    
    - Managed databases (AWS RDS, DigitalOcean)
    - Backup strategies
    - Migration strategies in production
    - Connection pooling
- **CI/CD Pipelines (1.5 hrs)**
    
    - GitHub Actions
    - Automated testing in CI
    - Automated deployment
    - Environment promotion
- **Monitoring in Production (1 hr)**
    
    - Uptime monitoring
    - Error tracking (Sentry)
    - Performance monitoring

**Time Breakdown:**

- Theory: 7 hrs
- Hands-on Setup & Deployment: 5-6 hrs

**Capstone Project:** Deploy Blog API to production with CI/CD, monitoring, and proper DevOps practices.

**Phase 6 Milestone:** Production-ready, maintainable system with proper testing and monitoring.

---

## 🎯 CAPSTONE PROJECT PROGRESSION

Build one comprehensive project across all 12 weeks, gradually adding features:

**Weeks 1-2:** Simple Todo API **Weeks 3-4:** Todo API with proper validation and testing **Weeks 5-6:** Blog API with users, posts, comments (database) **Weeks 7-8:** Add authentication, user roles, permissions **Weeks 9-10:** Add WebSockets for comments, real-time notifications, file uploads **Weeks 11-12:** Deploy with full test coverage, monitoring, CI/CD

### Capstone Final Feature List:

- ✅ User registration, login, JWT auth
- ✅ RBAC (Admin, Author, Reader roles)
- ✅ Posts, Comments, Categories with relationships
- ✅ Real-time chat/notifications via WebSockets
- ✅ File uploads (images, documents)
- ✅ Search and pagination
- ✅ Rate limiting
- ✅ 80%+ test coverage
- ✅ Structured logging
- ✅ Docker containerization
- ✅ Deployed to production with CI/CD
- ✅ Auto-generated API documentation

---

## 📚 RECOMMENDED LEARNING RESOURCES

### Official & Primary

1. **FastAPI Official Docs:** https://fastapi.tiangolo.com (best resource)
2. **Pydantic Docs:** https://docs.pydantic.dev
3. **SQLAlchemy Docs:** https://docs.sqlalchemy.org

### Courses

- **Full Stack FastAPI** (freeCodeCamp on YouTube) - 4 hours
- **FastAPI Tutorial Series** by Sebastián Ramírez (creator) - Official tutorials
- **Real Python FastAPI Tutorials** - High quality free articles

### Books

- "Building Data Science Applications with FastAPI" - Francesca Luca (optional, advanced)
- "SQLAlchemy 2.0 and Relationships" - Official documentation (reference)

### Practice Platforms

- Build along with YouTube tutorials
- **GitHub:** Study others' FastAPI projects
- **LeetCode/HackerRank:** Database design problems (weeks 5-6)

---

## 📋 WEEKLY CHECKLIST TEMPLATE

```
WEEK X CHECKLIST
[ ] Theory videos/reading completed
[ ] Practice exercises done
[ ] Checkpoint project started
[ ] Tests written for code
[ ] Code reviewed and refactored
[ ] Documentation written
[ ] Checkpoint project completed
[ ] Ready to move to next week
```

---

## 🔍 QUALITY CHECKPOINTS (Know when you're ready to move on)

**After Phase 1:** Can write decorators and async functions from memory; understand FastAPI's async nature

**After Phase 2:** Can build a complete API without database; understand validation and error handling deeply

**After Phase 3:** Can design database schemas; write complex queries; understand ORM relationships

**After Phase 4:** Can implement secure login systems; understand OAuth2; can explain JWT flow

**After Phase 5:** Can add real-time features; understand event-driven patterns; can implement background tasks

**After Phase 6:** Can deploy to production; write comprehensive tests; can troubleshoot production issues

---

## ⏱️ DAILY SCHEDULE SUGGESTION (for 20-30 hrs/week)

**Option 1: 4 hours daily (5 days/week)**

- 1.5 hrs: Learning (videos/reading)
- 2 hrs: Coding practice
- 0.5 hrs: Review & refactoring

**Option 2: 6 hours (3 days) + 4 hours (1 day)**

- Heavy coding days mid-week
- Lighter review day

**Option 3: Flexible (4-5 hrs when available)**

- Maintain consistency over perfection
- Batch learning (2-3 hrs video, then 2-3 hrs coding)

---

## ⚠️ COMMON PITFALLS TO AVOID

1. **Skipping decorator/async understanding** - Will haunt you later
2. **Not writing tests** - Leads to technical debt
3. **Ignoring database design** - Creates scalability issues
4. **Rushing authentication** - Security is critical
5. **Building in isolation** - Deploy early and often
6. **Not reading error messages** - FastAPI's are helpful
7. **Copy-pasting without understanding** - Defeats the purpose

---

## 🎓 PROGRESSION INDICATORS

|Stage|You Can...|
|---|---|
|**Week 2**|Build any simple API endpoint|
|**Week 4**|Handle complex validation and dependencies|
|**Week 6**|Design and query databases efficiently|
|**Week 8**|Secure production APIs with proper auth|
|**Week 10**|Build real-time, scalable features|
|**Week 12**|Deploy production systems independently|

---

## 💡 FINAL THOUGHTS

- **First 2 weeks:** Build strong foundations (don't rush)
- **Weeks 3-6:** Depth in core concepts (most important)
- **Weeks 7-10:** Real-world features (fun part)
- **Weeks 11-12:** Production reality (crucial)

**Success Metric:** By week 12, you should be able to:

- Build a production API from zero
- Deploy it with CI/CD
- Explain every part of your code
- Handle any FastAPI problem independently

Good luck! 🚀