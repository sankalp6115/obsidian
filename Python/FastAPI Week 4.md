**Duration:** 12-13 hours  
**Learning Style:** Formal definitions + Theory + Practical examples  
**Prerequisites:** Week 1-3 completed  
**Goal:** Master dependency injection, middleware, and request processing pipeline

---

## Table of Contents

1. [Advanced Dependency Injection - 2.5 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#advanced-dependency-injection---25-hours)
2. [Shared Dependencies & Sub-Dependencies - 3 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#shared-dependencies--sub-dependencies---3-hours)
3. [Database Sessions & Resource Management - 2.5 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#database-sessions--resource-management---25-hours)
4. [Middleware Concepts - 2.5 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#middleware-concepts---25-hours)
5. [CORS, Security Headers & Configuration - 1.5 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#cors-security-headers--configuration---15-hours)
6. [Week 4 Checkpoint Project](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#week-4-checkpoint-project)

---

# ADVANCED DEPENDENCY INJECTION - 2.5 HOURS

## Part 1: Dependency Injection Deep Dive

### Formal Definition: Dependency Injection (DI)

**Dependency Injection:** A software design pattern where a function or class receives (is "injected with") its dependencies rather than creating them itself. Dependencies are provided by an external entity (in FastAPI, the framework itself).

**Key principle:** Invert control - instead of a function controlling its dependencies, the framework controls and provides them.

### Understanding the Problem: Without Dependency Injection

```python
# ❌ PROBLEM: Function controls its own dependencies
class DatabaseConnection:
    def __init__(self, connection_string: str):
        self.connection_string = connection_string
    
    def query(self, sql: str):
        # Simulate query
        return f"Result from {self.connection_string}"

def get_user(user_id: int):
    """
    Function creates its own database connection
    Problems:
    1. Hard to test (creates real DB connection)
    2. Can't use different DB for testing
    3. Connection management mixed with business logic
    4. Creates new connection every call
    5. Can't reuse connection across endpoints
    """
    db = DatabaseConnection("postgresql://localhost")
    result = db.query(f"SELECT * FROM users WHERE id = {user_id}")
    return result

def get_posts(user_id: int):
    """Every function must create its own connection"""
    db = DatabaseConnection("postgresql://localhost")
    result = db.query(f"SELECT * FROM posts WHERE user_id = {user_id}")
    return result

# Code duplication! Each function needs to know how to create DB connection
```

### Understanding the Solution: With Dependency Injection

```python
# ✅ SOLUTION: Framework provides dependencies
def get_db_connection() -> DatabaseConnection:
    """
    Dependency function
    Framework calls this to get dependency
    Dependency is provided to functions that request it
    """
    return DatabaseConnection("postgresql://localhost")

def get_user(user_id: int, db: DatabaseConnection = Depends(get_db_connection)):
    """
    Function receives DB connection as parameter
    Framework calls get_db_connection() and passes result
    
    Benefits:
    1. Easy to test (inject mock DB)
    2. Can use different DB for testing
    3. Business logic separated from setup
    4. Framework manages connections
    5. Can reuse connection across multiple endpoints
    """
    result = db.query(f"SELECT * FROM users WHERE id = {user_id}")
    return result

def get_posts(user_id: int, db: DatabaseConnection = Depends(get_db_connection)):
    """Same dependency used in different endpoint"""
    result = db.query(f"SELECT * FROM posts WHERE user_id = {user_id}")
    return result

# No duplication! Dependency reused everywhere
```

### How FastAPI's Dependency System Works

**Formal Definition: Dependency Resolution** The process FastAPI uses to:

1. Identify which dependencies an endpoint needs
2. Resolve those dependencies (call dependency functions)
3. Handle dependency parameters
4. Inject resolved values into the endpoint

**Step-by-step flow:**

```python
from fastapi import FastAPI, Depends

app = FastAPI()

# Step 1: Define dependency
def get_current_user() -> str:
    """
    When FastAPI sees this function as dependency,
    it understands:
    - Function returns str
    - No parameters
    - Can be called anytime
    """
    return "alice"

# Step 2: Declare dependency in endpoint
@app.get("/users/me/")
def read_user(current_user: str = Depends(get_current_user)):
    """
    When request comes to /users/me/:
    
    1. FastAPI parses the function signature
    2. Finds parameter: current_user: str = Depends(get_current_user)
    3. Recognizes Depends() directive
    4. Calls get_current_user()
    5. Gets return value: "alice"
    6. Passes to function: current_user = "alice"
    7. Function executes and returns result
    """
    return {"current_user": current_user}

# Step 3: Request comes in
# GET /users/me/
# Response: {"current_user": "alice"}
```

---

## Part 2: Dependency Functions with Parameters

### Dependencies That Accept Parameters

```python
from fastapi import Depends, Query

def get_pagination_params(
    skip: int = Query(0, ge=0),
    limit: int = Query(10, ge=1, le=100)
) -> dict:
    """
    Dependency function that has its own parameters
    
    FastAPI automatically:
    1. Extracts skip and limit from query string
    2. Validates them
    3. Passes to this function
    4. Returns dict with pagination params
    """
    return {"skip": skip, "limit": limit}

@app.get("/items/")
def read_items(pagination: dict = Depends(get_pagination_params)):
    """
    When request comes with ?skip=20&limit=5:
    
    1. FastAPI calls get_pagination_params(skip=20, limit=5)
    2. Returns {"skip": 20, "limit": 5}
    3. Passes to read_items()
    
    URL: /items/?skip=20&limit=5
    Result: pagination = {"skip": 20, "limit": 5}
    """
    return {
        "items": [1, 2, 3],
        "pagination": pagination
    }
```

### Dependencies with Type-Checked Parameters

```python
from fastapi import Depends

class CommonQueryParams:
    """
    This is a dependency that's a class
    Every parameter becomes a class attribute
    """
    def __init__(
        self,
        skip: int = 0,
        limit: int = 10,
        search: str = None
    ):
        self.skip = skip
        self.limit = limit
        self.search = search

@app.get("/items/")
def read_items(params: CommonQueryParams = Depends(CommonQueryParams)):
    """
    Class dependency:
    1. FastAPI instantiates CommonQueryParams
    2. Passes query parameters to __init__
    3. Returns instance
    4. Passes instance to read_items()
    
    URL: /items/?skip=20&limit=5&search=widget
    params.skip = 20
    params.limit = 5
    params.search = "widget"
    """
    return {
        "items": [1, 2, 3],
        "skip": params.skip,
        "limit": params.limit,
        "search": params.search
    }
```

### Optional Dependencies

```python
from fastapi import Depends
from typing import Optional

def get_optional_token(token: Optional[str] = None) -> Optional[str]:
    """
    Dependency that's optional
    Returns None if not provided
    """
    return token

@app.get("/public/")
def public_endpoint(token: Optional[str] = Depends(get_optional_token)):
    """
    Token is optional
    
    URL: /public/ → token = None
    URL: /public/?token=abc123 → token = "abc123"
    """
    if token:
        return {"message": "Logged in", "token": token}
    return {"message": "Public access"}
```

---

## Part 3: Dependency Return Types and Validation

### Typed Dependencies

```python
from fastapi import Depends
from pydantic import BaseModel

class User(BaseModel):
    """Pydantic model for type safety"""
    id: int
    name: str
    email: str

def get_current_user() -> User:
    """
    Dependency that returns typed object
    FastAPI understands return type is User
    """
    return User(id=1, name="Alice", email="alice@example.com")

@app.get("/users/me/", response_model=User)
def read_current_user(current_user: User = Depends(get_current_user)):
    """
    current_user is guaranteed to be User instance
    Fully type-checked
    """
    return current_user
```

### Complex Dependency Return Types

```python
from fastapi import Depends
from typing import Dict, List

def get_user_data() -> Dict[str, str]:
    """Return typed dictionary"""
    return {"id": "1", "name": "Alice"}

def get_user_permissions() -> List[str]:
    """Return typed list"""
    return ["read", "write", "delete"]

@app.get("/dashboard/")
def show_dashboard(
    user_data: Dict[str, str] = Depends(get_user_data),
    permissions: List[str] = Depends(get_user_permissions)
):
    """
    Multiple dependencies with different return types
    All type-checked automatically
    """
    return {
        "user": user_data,
        "permissions": permissions
    }
```

---

# SHARED DEPENDENCIES & SUB-DEPENDENCIES - 3 HOURS

## Part 1: Dependency Reuse Across Endpoints

### Formal Definition: Dependency Reuse

**Dependency Reuse:** Using the same dependency function in multiple endpoints. Enables code reuse and ensures consistency across the application.

```python
from fastapi import Depends

def get_current_user() -> str:
    """
    This dependency is used in multiple endpoints
    Ensures all endpoints use same authentication logic
    """
    return "alice"

def check_permissions(current_user: str) -> bool:
    """
    This dependency uses current_user
    Creates chain of dependencies
    """
    return current_user == "alice"

# Endpoint 1: Uses get_current_user
@app.get("/users/me/")
def read_current_user(current_user: str = Depends(get_current_user)):
    return {"current_user": current_user}

# Endpoint 2: Uses same dependency
@app.get("/posts/")
def read_posts(current_user: str = Depends(get_current_user)):
    return {"posts": [], "user": current_user}

# Endpoint 3: Uses same dependency
@app.get("/settings/")
def read_settings(current_user: str = Depends(get_current_user)):
    return {"settings": {}, "user": current_user}

# All three endpoints execute get_current_user()
# Ensures consistent authentication across all endpoints
```

### DRY Principle with Dependencies

```python
# ❌ WITHOUT dependency - code duplication
@app.get("/items/")
def read_items(skip: int = 0, limit: int = 10):
    """Manual pagination"""
    return {"items": [1, 2, 3], "skip": skip, "limit": limit}

@app.get("/posts/")
def read_posts(skip: int = 0, limit: int = 10):
    """Same pagination logic repeated"""
    return {"posts": [], "skip": skip, "limit": limit}

# ✅ WITH dependency - no duplication
def get_pagination(skip: int = 0, limit: int = 10) -> dict:
    """Pagination logic in one place"""
    return {"skip": skip, "limit": limit}

@app.get("/items/")
def read_items(pagination: dict = Depends(get_pagination)):
    """Uses shared pagination"""
    return {"items": [1, 2, 3], **pagination}

@app.get("/posts/")
def read_posts(pagination: dict = Depends(get_pagination)):
    """Same dependency"""
    return {"posts": [], **pagination}

# If pagination logic changes, only one place to update!
```

---

## Part 2: Sub-Dependencies (Dependency Chains)

### Formal Definition: Sub-Dependencies

**Sub-Dependency:** A dependency that itself depends on other dependencies. Creates a chain where one dependency requires another.

**Dependency chain:** A → B → C Where A depends on B, and B depends on C.

### Simple Dependency Chain

```python
from fastapi import Depends

# Level 3: Base dependency
def get_database() -> str:
    """Lowest level dependency - no dependencies"""
    return "postgresql://localhost"

# Level 2: Depends on Level 3
def get_current_user(db: str = Depends(get_database)) -> str:
    """
    This dependency uses get_database
    Creates a chain:
    get_current_user → get_database
    """
    # Would use db to fetch current user
    return "alice"

# Level 1: Depends on Level 2
@app.get("/users/me/")
def read_current_user(
    current_user: str = Depends(get_current_user)
):
    """
    Endpoint depends on current_user
    which depends on get_database
    
    Dependency resolution:
    1. Endpoint needs current_user
    2. current_user needs db
    3. FastAPI calls get_database()
    4. Passes result to get_current_user()
    5. Passes result to endpoint
    """
    return {"current_user": current_user}
```

**Visual representation:**

```
Request to /users/me/
    ↓
Endpoint function needs: current_user
    ↓
FastAPI sees: Depends(get_current_user)
    ↓
get_current_user() needs: db
    ↓
FastAPI sees: Depends(get_database)
    ↓
FastAPI calls get_database()
    ↓
get_database() returns: "postgresql://localhost"
    ↓
FastAPI calls get_current_user(db="postgresql://localhost")
    ↓
get_current_user() returns: "alice"
    ↓
FastAPI calls read_current_user(current_user="alice")
    ↓
Endpoint returns: {"current_user": "alice"}
```

### Complex Dependency Chains

```python
from fastapi import Depends, HTTPException

# Base: Database connection
def get_db() -> str:
    return "db_connection"

# Level 2: Token validation
def verify_token(token: str, db: str = Depends(get_db)) -> str:
    """
    Depends on: get_db
    Uses both token parameter and db dependency
    """
    if token == "valid":
        return token
    raise HTTPException(status_code=401, detail="Invalid token")

# Level 3: User authentication
def get_current_user(
    token: str = None,
    db: str = Depends(get_db),
    verified_token: str = Depends(verify_token)
) -> str:
    """
    Depends on: get_db, verify_token
    verified_token depends on get_db
    
    Dependency tree:
    get_current_user
    ├── get_db
    ├── verify_token
    │   └── get_db (same instance!)
    """
    return "alice"

# Level 4: Permissions check
def check_admin(current_user: str = Depends(get_current_user)) -> bool:
    """
    Depends on: get_current_user
    which depends on verify_token
    which depends on get_db
    
    Full chain: admin → current_user → verify_token → db
    """
    return current_user == "admin"

# Endpoint uses the full chain
@app.get("/admin/")
def admin_dashboard(is_admin: bool = Depends(check_admin)):
    """
    When this endpoint is called:
    1. FastAPI resolves is_admin dependency
    2. Which requires check_admin
    3. Which requires get_current_user
    4. Which requires verify_token
    5. Which requires get_db
    
    FastAPI automatically resolves entire chain!
    And reuses instances (get_db called once, result used multiple times)
    """
    return {"admin": is_admin}
```

---

## Part 3: Dependency Caching and Reuse

### Formal Definition: Dependency Caching

**Dependency Caching:** When the same dependency is used multiple times in a request, FastAPI caches the result and reuses it instead of calling the function multiple times.

### Dependency Reuse Within Single Request

```python
from fastapi import Depends

call_count = 0

def get_current_user() -> str:
    """
    This function will be called only ONCE per request
    even if multiple endpoints/dependencies need it
    """
    global call_count
    call_count += 1
    print(f"get_current_user called (call #{call_count})")
    return "alice"

def get_user_permissions(current_user: str = Depends(get_current_user)):
    """
    Needs current_user
    get_current_user already called above?
    FastAPI reuses cached result!
    """
    return ["read", "write"]

@app.get("/dashboard/")
def dashboard(
    current_user: str = Depends(get_current_user),  # Call 1
    permissions: list = Depends(get_user_permissions)  # Uses cached result
):
    """
    When request comes to /dashboard/:
    
    1. FastAPI processes dependencies
    2. Needs current_user → calls get_current_user()
    3. Needs permissions from get_user_permissions
    4. get_user_permissions needs current_user
    5. FastAPI finds it's already resolved → reuses result!
    6. Passes cached "alice" to get_user_permissions
    
    Output:
    get_current_user called (call #1)
    
    Only 1 call! Not 2!
    """
    return {
        "user": current_user,
        "permissions": permissions
    }
```

### When Caching Matters

```python
from fastapi import Depends

# Expensive operation
def get_current_user_from_db() -> str:
    """
    Expensive operation (database query, API call, etc.)
    Only want to call once per request!
    """
    print("Expensive DB query...")
    return "alice"

def get_user_data(current_user: str = Depends(get_current_user_from_db)):
    """Uses current_user"""
    return {"name": current_user}

def get_user_permissions(current_user: str = Depends(get_current_user_from_db)):
    """Also uses current_user"""
    return ["read", "write"]

def get_user_settings(current_user: str = Depends(get_current_user_from_db)):
    """Also uses current_user"""
    return {"theme": "dark"}

@app.get("/profile/")
def profile(
    data: dict = Depends(get_user_data),
    permissions: list = Depends(get_user_permissions),
    settings: dict = Depends(get_user_settings)
):
    """
    Three dependencies all need current_user
    
    Output:
    Expensive DB query...
    
    Only 1 query! Cached and reused 3 times!
    Without caching, would query DB 3 times!
    """
    return {"data": data, "permissions": permissions, "settings": settings}
```

---

# DATABASE SESSIONS & RESOURCE MANAGEMENT - 2.5 HOURS

## Part 1: Context Managers in Dependencies

### Formal Definition: Context Manager in Dependencies

**Context Manager:** An object that manages resources (like database connections) ensuring they're properly set up and cleaned up. Used with `yield` in dependencies.

```python
from fastapi import Depends
from typing import Generator

# ❌ WITHOUT context manager - resource leak
def get_db():
    """
    Creates connection but never closes it!
    Connection stays open after endpoint finishes
    Eventually run out of connections
    """
    db = create_connection("postgresql://localhost")
    return db

# ✅ WITH context manager - proper cleanup
def get_db() -> Generator:
    """
    Creates connection
    Yields it to endpoint
    Closes it when endpoint finishes
    
    Uses 'yield' to pause execution
    """
    db = create_connection("postgresql://localhost")
    try:
        yield db  # Give connection to endpoint
    finally:
        db.close()  # Always cleanup, even if error

@app.get("/items/")
def read_items(db = Depends(get_db)):
    """
    When endpoint uses this dependency:
    
    1. FastAPI calls get_db()
    2. Executes up to 'yield'
    3. Connection created and yields db
    4. Endpoint receives db
    5. Endpoint executes
    6. Endpoint finishes
    7. Execution continues after 'yield'
    8. db.close() called (cleanup)
    """
    result = db.query("SELECT * FROM items")
    return result
```

### Yield Execution Flow

**Visual representation:**

```
get_db() function:
┌─────────────────────────────────────┐
│ db = create_connection(...)         │  Endpoint starts
│ try:                                │
│     yield db  ← PAUSE HERE ←────────┼─ Give db to endpoint
│              ↑ RESUME HERE ←────────┼─ Endpoint finishes
│ finally:                            │
│     db.close()                      │  Cleanup happens
└─────────────────────────────────────┘
```

### Dependency Yield for Resource Management

```python
from fastapi import Depends
from typing import Generator

class Database:
    def __init__(self, url: str):
        self.url = url
        self.connection = None
    
    def connect(self):
        print(f"Connecting to {self.url}")
        self.connection = f"Connection to {self.url}"
    
    def close(self):
        print(f"Closing connection")
        self.connection = None

def get_database() -> Generator:
    """
    Dependency that manages database lifecycle
    """
    db = Database("postgresql://localhost")
    db.connect()
    try:
        yield db
    finally:
        db.close()

@app.get("/items/")
def read_items(db: Database = Depends(get_database)):
    """
    When request comes:
    
    Output:
    Connecting to postgresql://localhost
    [endpoint executes]
    Closing connection
    
    Connection opened, used, and closed automatically
    """
    return {"items": []}
```

### Error Handling in Context Managers

```python
from fastapi import Depends

def get_db() -> Generator:
    """
    Even if endpoint raises error,
    cleanup code still runs
    """
    db = create_connection()
    try:
        yield db
    except Exception as e:
        # Handle database errors
        print(f"Error in endpoint: {e}")
        db.rollback()  # Rollback transaction
        raise
    finally:
        # ALWAYS runs, regardless of error
        db.close()

@app.get("/items/")
def read_items(db = Depends(get_db)):
    """
    If this raises error:
    
    1. get_db catches it
    2. Rolls back transaction
    3. Re-raises error
    4. Finally block runs
    5. Connection closes
    """
    raise ValueError("Oops!")
```

---

## Part 2: Database Session Management Pattern

### Formal Definition: Session Management

**Session:** A conversation with the database that maintains state (like open transaction). Proper session management ensures:

1. Sessions created once per request
2. Same session used by all dependencies
3. Session closed/cleaned up after request
4. Errors rolled back automatically

### SQLAlchemy Session Dependency

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, Session
from fastapi import Depends
from typing import Generator

# Create engine (connection pool)
engine = create_engine("postgresql://user:password@localhost/dbname")

# Create SessionLocal class
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

def get_db() -> Generator[Session, None, None]:
    """
    Dependency that provides database session
    
    Type hints:
    - Generator[Session, None, None]
      - Yields: Session object
      - Send: None (not using send)
      - Return: None (not returning value)
    """
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# Now use in endpoints
@app.get("/users/")
def read_users(db: Session = Depends(get_db)):
    """
    Each request:
    1. Creates new session
    2. Passes to endpoint
    3. Endpoint uses for queries
    4. Session closes after endpoint
    """
    users = db.query(User).all()
    return users

@app.post("/users/")
def create_user(user: UserCreate, db: Session = Depends(get_db)):
    """Create user in database"""
    db_user = User(name=user.name, email=user.email)
    db.add(db_user)
    db.commit()  # Commit transaction
    db.refresh(db_user)  # Refresh to get DB-generated ID
    return db_user
```

### Sub-Dependencies with Sessions

```python
from sqlalchemy.orm import Session

def get_current_user(db: Session = Depends(get_db)) -> User:
    """
    This dependency needs a session
    get_db provides it
    """
    user = db.query(User).filter(User.id == 1).first()
    return user

def get_user_posts(
    current_user: User = Depends(get_current_user),
    db: Session = Depends(get_db)
) -> List[Post]:
    """
    Depends on both current_user and db
    
    But get_current_user already used same db session!
    FastAPI reuses cached session instance
    So there's only ONE session for entire request chain
    """
    posts = db.query(Post).filter(
        Post.user_id == current_user.id
    ).all()
    return posts

@app.get("/users/me/posts/")
def my_posts(posts: List[Post] = Depends(get_user_posts)):
    """
    Request flow:
    1. Need posts from get_user_posts
    2. get_user_posts needs current_user and db
    3. get_current_user needs db
    4. FastAPI calls get_db() once
    5. All dependencies share the same session instance!
    6. After endpoint finishes, session closes
    
    ONE session, used across entire dependency chain
    """
    return posts
```

---

## Part 3: Multiple Database Sessions

```python
from fastapi import Depends

# Multiple database connections
engine_primary = create_engine("postgresql://primary-db")
engine_cache = create_engine("postgresql://cache-db")

SessionPrimary = sessionmaker(bind=engine_primary)
SessionCache = sessionmaker(bind=engine_cache)

def get_primary_db() -> Generator:
    """Primary database session"""
    db = SessionPrimary()
    try:
        yield db
    finally:
        db.close()

def get_cache_db() -> Generator:
    """Cache database session"""
    db = SessionCache()
    try:
        yield db
    finally:
        db.close()

@app.get("/items/")
def read_items(
    primary_db: Session = Depends(get_primary_db),
    cache_db: Session = Depends(get_cache_db)
):
    """
    Use different databases:
    - Query from cache first
    - Fall back to primary
    - Write to primary
    """
    # Check cache
    cached = cache_db.query(Item).first()
    if cached:
        return cached
    
    # Query primary
    item = primary_db.query(Item).first()
    
    # Cache result
    cache_db.add(item)
    cache_db.commit()
    
    return item
```

---

# MIDDLEWARE CONCEPTS - 2.5 HOURS

## Part 1: Understanding Middleware

### Formal Definition: Middleware

**Middleware:** Software that processes requests before they reach the endpoint, and responses before they're sent to the client. Middleware sits in the middle of the request/response cycle.

**Middleware functions:**

- Run before endpoint (request processing)
- Run after endpoint (response processing)
- Can modify requests or responses
- Can block requests from reaching endpoint
- Can add headers, validate, etc.

### Request/Response Lifecycle with Middleware

```
Client sends request
    ↓
┌─────────────────────────────────────┐
│ Middleware 1 - Request              │ ← Runs first
│ (e.g., logging, authentication)    │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│ Middleware 2 - Request              │ ← Runs second
│ (e.g., CORS, headers)              │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│ Route/Endpoint Handler              │ ← Processes request
│ (your business logic)               │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│ Middleware 2 - Response             │ ← Runs in reverse
│ (e.g., CORS, headers)              │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│ Middleware 1 - Response             │ ← Runs last
│ (e.g., logging)                    │
└─────────────────────────────────────┘
    ↓
Response sent to client
```

### Creating Simple Middleware

```python
from fastapi import FastAPI
from starlette.middleware.base import BaseHTTPMiddleware
from starlette.requests import Request
import time

app = FastAPI()

class ProcessTimeMiddleware(BaseHTTPMiddleware):
    """
    Middleware that adds processing time to response header
    """
    async def dispatch(self, request: Request, call_next):
        """
        dispatch() is called for every request
        
        request: The incoming request
        call_next: Function to call next middleware/endpoint
        """
        # Runs BEFORE endpoint
        start_time = time.time()
        
        # Call next middleware or endpoint
        response = await call_next(request)
        
        # Runs AFTER endpoint
        process_time = time.time() - start_time
        
        # Add header to response
        response.headers["X-Process-Time"] = str(process_time)
        
        return response

# Add middleware to app
app.add_middleware(ProcessTimeMiddleware)

@app.get("/items/")
def read_items():
    """
    When request comes:
    1. ProcessTimeMiddleware.dispatch() called
    2. Measures start time
    3. Calls await call_next(request)
    4. Endpoint executes
    5. Response created
    6. Middleware adds X-Process-Time header
    7. Returns response to client
    """
    return {"items": []}

# Response will include header:
# X-Process-Time: 0.0123
```

### Middleware Order

```python
# ❌ WRONG ORDER
app.add_middleware(TimingMiddleware)
app.add_middleware(LoggingMiddleware)

# Request flow:
# LoggingMiddleware → TimingMiddleware → Endpoint
# (Reverse of add order)

# ✅ CORRECT ORDER
app.add_middleware(LoggingMiddleware)
app.add_middleware(TimingMiddleware)

# Request flow:
# TimingMiddleware → LoggingMiddleware → Endpoint
# (Reverse of add order)

# Remember: Middleware added later runs first!
# This is counterintuitive but correct
```

---

## Part 2: Practical Middleware Examples

### Logging Middleware

```python
from starlette.middleware.base import BaseHTTPMiddleware
from starlette.requests import Request
import logging

logger = logging.getLogger(__name__)

class LoggingMiddleware(BaseHTTPMiddleware):
    """
    Logs all requests and responses
    """
    async def dispatch(self, request: Request, call_next):
        # Log request
        logger.info(f"Request: {request.method} {request.url}")
        logger.info(f"Headers: {dict(request.headers)}")
        
        # Call endpoint
        response = await call_next(request)
        
        # Log response
        logger.info(f"Response Status: {response.status_code}")
        
        return response

app.add_middleware(LoggingMiddleware)
```

### Authentication Middleware

```python
from starlette.middleware.base import BaseHTTPMiddleware
from starlette.requests import Request
from starlette.responses import JSONResponse

class AuthenticationMiddleware(BaseHTTPMiddleware):
    """
    Validates authentication token in all requests
    """
    async def dispatch(self, request: Request, call_next):
        # Get token from header
        token = request.headers.get("Authorization")
        
        # If no token, return 401
        if not token:
            return JSONResponse(
                status_code=401,
                content={"detail": "Missing authentication token"}
            )
        
        # Validate token
        if not self.validate_token(token):
            return JSONResponse(
                status_code=401,
                content={"detail": "Invalid token"}
            )
        
        # Token valid, continue to endpoint
        response = await call_next(request)
        return response
    
    def validate_token(self, token: str) -> bool:
        """Check if token is valid"""
        return token.startswith("Bearer ")

app.add_middleware(AuthenticationMiddleware)
```

### Request Modification Middleware

```python
from starlette.middleware.base import BaseHTTPMiddleware

class RequestIDMiddleware(BaseHTTPMiddleware):
    """
    Adds unique request ID to each request
    Helpful for tracing requests in logs
    """
    async def dispatch(self, request: Request, call_next):
        import uuid
        
        # Generate unique ID for this request
        request_id = str(uuid.uuid4())
        
        # Add to request (as custom attribute)
        request.state.request_id = request_id
        
        # Add to response header
        response = await call_next(request)
        response.headers["X-Request-ID"] = request_id
        
        return response

# In endpoint, access request ID
@app.get("/items/")
def read_items(request: Request):
    """Access request ID added by middleware"""
    request_id = request.state.request_id
    return {"items": [], "request_id": request_id}
```

---

## Part 3: Built-in Middleware

### CORS Middleware

**Formal Definition: CORS (Cross-Origin Resource Sharing)** A security mechanism that controls which websites can access your API. Browser enforces CORS to prevent unauthorized access.

```python
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

# Configure CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "http://localhost:3000",  # React dev server
        "http://localhost:8080",  # Vue dev server
        "https://example.com"     # Production frontend
    ],
    allow_credentials=True,  # Allow cookies/credentials
    allow_methods=["*"],     # Allow all HTTP methods
    allow_headers=["*"]      # Allow all headers
)

# Alternatively, allow all origins (NOT recommended for production)
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # Allow ANY origin
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"]
)
```

### Trusted Host Middleware

```python
from starlette.middleware.trustedhost import TrustedHostMiddleware

app.add_middleware(
    TrustedHostMiddleware,
    allowed_hosts=[
        "example.com",
        "www.example.com",
        "api.example.com"
    ]
)

# Only these hosts can access API
# Prevents Host header attacks
```

### HTTPS Redirect Middleware

```python
from starlette.middleware.https import HTTPSMiddleware

app.add_middleware(HTTPSMiddleware)

# Forces all HTTP requests to redirect to HTTPS
# http://example.com/items → https://example.com/items
```

---

# CORS, SECURITY HEADERS & CONFIGURATION - 1.5 HOURS

## Part 1: CORS Deep Dive

### Formal Definition: CORS Attacks and Why Needed

**Same-Origin Policy:** Browser security feature that prevents scripts from one website from accessing data from another website without permission.

**The Problem:**

```javascript
// On attacker.com, running JavaScript
fetch("https://yourbank.com/transfer-money", {
    method: "POST",
    body: {to: "attacker", amount: 1000000}
})
// Without CORS, this could work if user is logged in!
// Browser would send cookies and transfer money!
```

**CORS Solution:** Your API specifies which origins are allowed to access it. Browser checks and blocks unauthorized requests.

### CORS Configuration Levels

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

# Level 1: No CORS (only same origin)
# Don't add middleware - browser enforces same-origin policy

# Level 2: Allow specific origins (RECOMMENDED for production)
app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "https://example.com",  # Exact matches only
        "https://www.example.com",
        "https://admin.example.com"
    ],
    allow_credentials=True,
    allow_methods=["GET", "POST"],  # Specific methods
    allow_headers=["Content-Type", "Authorization"],  # Specific headers
    max_age=600  # Browser caches CORS response for 10 minutes
)

# Level 3: Allow all origins (NOT recommended - security risk)
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # Anyone can access
    allow_credentials=True,  # But this contradicts allow_origins=["*"]
    allow_methods=["*"],
    allow_headers=["*"]
)

# Level 4: Allow origins from environment (for different environments)
import os

allow_origins = os.getenv(
    "ALLOWED_ORIGINS",
    "http://localhost:3000"  # Default for development
).split(",")

app.add_middleware(
    CORSMiddleware,
    allow_origins=allow_origins,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"]
)
```

### CORS Preflight Requests

**Formal Definition: Preflight Request** An automatic request sent by browser before actual request to check if API allows the cross-origin request.

```
User clicks button in frontend (origin: localhost:3000)
    ↓
Browser needs to send POST to API (origin: localhost:8000)
    ↓
Different origin! Browser sends PREFLIGHT request first:
    
OPTIONS /items/ HTTP/1.1
Origin: http://localhost:3000
Access-Control-Request-Method: POST
Access-Control-Request-Headers: content-type
    ↓
API responds with CORS headers:
    
HTTP/1.1 200 OK
Access-Control-Allow-Origin: http://localhost:3000
Access-Control-Allow-Methods: GET, POST
Access-Control-Allow-Headers: content-type
    ↓
Browser sees response and says: "OK, this origin is allowed"
    ↓
Browser sends actual POST request
    ↓
POST /items/ HTTP/1.1
[actual request body]
```

---

## Part 2: Security Headers

### Formal Definition: Security Headers

**Security Headers:** HTTP headers that instruct browsers how to handle your website/API. Provide defense against common attacks.

### Common Security Headers

```python
from fastapi import FastAPI
from starlette.middleware.base import BaseHTTPMiddleware
from starlette.requests import Request

app = FastAPI()

class SecurityHeadersMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        response = await call_next(request)
        
        # 1. X-Content-Type-Options
        # Prevents MIME type sniffing attacks
        response.headers["X-Content-Type-Options"] = "nosniff"
        
        # 2. X-Frame-Options
        # Prevents clickjacking attacks
        # DENY: Can't be framed at all
        # SAMEORIGIN: Can only be framed by same site
        response.headers["X-Frame-Options"] = "DENY"
        
        # 3. X-XSS-Protection
        # Enables browser XSS protection
        response.headers["X-XSS-Protection"] = "1; mode=block"
        
        # 4. Strict-Transport-Security (HSTS)
        # Forces HTTPS for all future requests
        # max-age in seconds: 31536000 = 1 year
        response.headers["Strict-Transport-Security"] = "max-age=31536000; includeSubDomains"
        
        # 5. Content-Security-Policy (CSP)
        # Controls which resources can be loaded
        response.headers["Content-Security-Policy"] = "default-src 'self'"
        
        # 6. Referrer-Policy
        # Controls referrer information sent
        response.headers["Referrer-Policy"] = "no-referrer"
        
        return response

app.add_middleware(SecurityHeadersMiddleware)

# Or use built-in middleware
from starlette.middleware.hsts import HSTSMiddleware

app.add_middleware(
    HSTSMiddleware,
    max_age=31536000,  # 1 year
    include_subdomains=True
)
```

---

## Part 3: Configuration Best Practices

### Environment-Based Configuration

```python
import os
from fastapi import FastAPI

app = FastAPI()

# Get environment
ENVIRONMENT = os.getenv("ENVIRONMENT", "development")

# Configure based on environment
if ENVIRONMENT == "production":
    # Production: strict CORS
    allowed_origins = [
        "https://example.com",
        "https://www.example.com"
    ]
    # Production: all security headers
    enable_security_headers = True
    debug_enabled = False
    
elif ENVIRONMENT == "staging":
    # Staging: looser CORS
    allowed_origins = [
        "https://staging.example.com",
        "https://localhost:3000"
    ]
    enable_security_headers = True
    debug_enabled = False
    
else:  # development
    # Development: allow all for testing
    allowed_origins = ["*"]
    enable_security_headers = False
    debug_enabled = True

# Apply configuration
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=allowed_origins,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"]
)

if enable_security_headers:
    # Add security headers only in production/staging
    class SecurityHeadersMiddleware(BaseHTTPMiddleware):
        async def dispatch(self, request: Request, call_next):
            response = await call_next(request)
            response.headers["X-Content-Type-Options"] = "nosniff"
            response.headers["X-Frame-Options"] = "DENY"
            return response
    
    app.add_middleware(SecurityHeadersMiddleware)
```

### Configuration from Environment File

```python
# .env file
ENVIRONMENT=production
ALLOWED_ORIGINS=https://example.com,https://www.example.com
DATABASE_URL=postgresql://user:pass@localhost/dbname
SECRET_KEY=your-secret-key-here

# main.py
from pydantic import BaseSettings
import os

class Settings(BaseSettings):
    """
    Load configuration from environment variables
    """
    environment: str = "development"
    allowed_origins: list = ["*"]
    database_url: str
    secret_key: str
    
    class Config:
        env_file = ".env"  # Load from .env file

settings = Settings()

from fastapi.middleware.cors import CORSMiddleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.allowed_origins,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"]
)
```

---

# WEEK 4 CHECKPOINT PROJECT

## Project: Blog API with Authentication & Authorization

### Objective

Build a complete Blog API featuring:

1. Advanced dependency injection
2. Database session management
3. Authentication via dependencies
4. Role-based authorization
5. Middleware for logging and security
6. CORS configuration
7. Complete request/response validation

### Project Requirements

#### 1. Data Models

```python
class User(BaseModel):
    id: int
    username: str = Field(..., min_length=3, max_length=50)
    email: str
    full_name: str
    role: str = "user"  # user, editor, admin
    is_active: bool = True

class UserCreate(BaseModel):
    username: str
    email: str
    full_name: str
    password: str = Field(..., min_length=8)

class Post(BaseModel):
    id: int
    title: str = Field(..., min_length=5, max_length=200)
    content: str
    author_id: int
    published: bool = False
    created_at: str

class PostCreate(BaseModel):
    title: str
    content: str
```

#### 2. Dependencies to Create

**Dependency 1: Database Session**

```python
def get_db() -> Generator:
    """Provide database session for each request"""
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

**Dependency 2: Current User Authentication**

```python
def get_current_user(
    token: str = Header(...),
    db: Session = Depends(get_db)
) -> User:
    """
    Validate JWT token and get current user
    Required for protected endpoints
    """
    # Validate token
    # Get user from database
    return user
```

**Dependency 3: Role-Based Authorization**

```python
def check_admin(
    current_user: User = Depends(get_current_user)
) -> User:
    """
    Verify current user is admin
    Only admins can access certain endpoints
    """
    if current_user.role != "admin":
        raise HTTPException(status_code=403)
    return current_user

def check_editor_or_admin(
    current_user: User = Depends(get_current_user)
) -> User:
    """
    Verify current user is editor or admin
    """
    if current_user.role not in ["editor", "admin"]:
        raise HTTPException(status_code=403)
    return current_user
```

**Dependency 4: Post Ownership**

```python
def get_post_if_owner(
    post_id: int = Path(...),
    current_user: User = Depends(get_current_user),
    db: Session = Depends(get_db)
) -> Post:
    """
    Get post and verify current user is author
    """
    post = db.query(Post).filter(Post.id == post_id).first()
    if not post:
        raise HTTPException(status_code=404)
    if post.author_id != current_user.id:
        raise HTTPException(status_code=403)
    return post
```

#### 3. API Endpoints

**Public Endpoints:**

1. POST /register - Create new user
2. POST /login - Get authentication token
3. GET /posts/ - List published posts
4. GET /posts/{id} - Get specific post

**Protected Endpoints (require authentication):** 5. GET /users/me - Current user profile 6. POST /posts/ - Create new post (editors+) 7. PUT /posts/{id} - Update post (owner or admin) 8. DELETE /posts/{id} - Delete post (owner or admin)

**Admin Endpoints:** 9. GET /admin/users - List all users 10. PUT /admin/users/{id}/role - Change user role

#### 4. Middleware to Create

**Middleware 1: Request Logging**

```python
class LoggingMiddleware(BaseHTTPMiddleware):
    """Log all requests and responses"""
    async def dispatch(self, request: Request, call_next):
        logger.info(f"{request.method} {request.url}")
        response = await call_next(request)
        logger.info(f"Status: {response.status_code}")
        return response
```

**Middleware 2: Security Headers**

```python
class SecurityHeadersMiddleware(BaseHTTPMiddleware):
    """Add security headers to all responses"""
    async def dispatch(self, request: Request, call_next):
        response = await call_next(request)
        response.headers["X-Content-Type-Options"] = "nosniff"
        response.headers["X-Frame-Options"] = "DENY"
        return response
```

**Middleware 3: Request Timing**

```python
class ProcessTimeMiddleware(BaseHTTPMiddleware):
    """Add processing time to response header"""
    async def dispatch(self, request: Request, call_next):
        start = time.time()
        response = await call_next(request)
        response.headers["X-Process-Time"] = str(time.time() - start)
        return response
```

#### 5. Configuration

```python
# CORS configuration - allow frontend origin
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"]
)

# Add custom middleware
app.add_middleware(ProcessTimeMiddleware)
app.add_middleware(SecurityHeadersMiddleware)
app.add_middleware(LoggingMiddleware)
```

#### 6. JWT Token Handling

```python
import jwt
from datetime import datetime, timedelta

SECRET_KEY = "your-secret-key"
ALGORITHM = "HS256"

def create_access_token(user_id: int) -> str:
    """Create JWT token for user"""
    payload = {
        "user_id": user_id,
        "exp": datetime.utcnow() + timedelta(hours=24)
    }
    return jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)

def verify_token(token: str) -> int:
    """Verify token and return user_id"""
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        return payload["user_id"]
    except:
        return None
```

### Deliverables

1. **main.py** - FastAPI application with all endpoints
2. **dependencies.py** - All dependency functions
3. **middleware.py** - Custom middleware classes
4. **models.py** - Pydantic models
5. **database.py** - Database configuration and session management
6. **auth.py** - Authentication and token handling
7. **README.md** - Complete documentation
8. **requirements.txt** - All dependencies

### Testing Checklist

- [ ] Register new user
- [ ] Login and get token
- [ ] Access protected endpoint with token
- [ ] 401 error without token
- [ ] 403 error with insufficient permissions
- [ ] Create post (requires authentication)
- [ ] Update own post (success)
- [ ] Update others' post (403 error)
- [ ] Admin can update any post
- [ ] All endpoints log requests
- [ ] Security headers present in responses
- [ ] CORS works with frontend origin

---

# EXERCISE SOLUTIONS

## Dependency Injection Exercises

### Exercise 1: Pagination Dependency

**Exercise:**

```python
# Create a dependency that handles pagination
# - skip: default 0
# - limit: default 10, max 100
# - Use it in GET /items/ endpoint
```

**Solution:**

```python
from fastapi import Query, Depends

def get_pagination(
    skip: int = Query(0, ge=0),
    limit: int = Query(10, ge=1, le=100)
) -> dict:
    return {"skip": skip, "limit": limit}

@app.get("/items/")
def read_items(pagination: dict = Depends(get_pagination)):
    return {
        "items": [1, 2, 3],
        "pagination": pagination
    }
```

### Exercise 2: Authentication Chain

**Exercise:**

```python
# Create dependency chain:
# get_token → validate_token → get_current_user
```

**Solution:**

```python
def get_token(authorization: str = Header(...)) -> str:
    return authorization.replace("Bearer ", "")

def validate_token(token: str = Depends(get_token)) -> str:
    if not token.startswith("valid"):
        raise HTTPException(status_code=401)
    return token

def get_current_user(
    validated_token: str = Depends(validate_token)
) -> str:
    return "alice"

@app.get("/me/")
def get_me(user: str = Depends(get_current_user)):
    return {"user": user}
```

---

# SUMMARY AND CHECKLIST

## Week 4 Learning Path Summary

**Total Time: 12-13 Hours**

### Advanced Dependency Injection (2.5 hours)

- ✅ Dependency injection pattern and benefits
- ✅ Dependencies with parameters
- ✅ Class-based dependencies
- ✅ Optional dependencies
- ✅ Type-checked dependencies

### Shared Dependencies & Sub-Dependencies (3 hours)

- ✅ Dependency reuse across endpoints
- ✅ Dependency chains (A → B → C)
- ✅ Dependency caching within request
- ✅ Reusing cached results
- ✅ Complex dependency trees

### Database Sessions & Resource Management (2.5 hours)

- ✅ Context managers with yield
- ✅ Resource cleanup (try/finally)
- ✅ Database session management
- ✅ SQLAlchemy session dependency
- ✅ Multiple database connections
- ✅ Transaction management

### Middleware Concepts (2.5 hours)

- ✅ Request/response lifecycle
- ✅ Creating custom middleware
- ✅ Middleware order
- ✅ Logging middleware
- ✅ Authentication middleware
- ✅ Request modification

### CORS & Security Headers (1.5 hours)

- ✅ CORS configuration
- ✅ Preflight requests
- ✅ Allowed origins
- ✅ Security headers
- ✅ Environment-based configuration
- ✅ HTTPS enforcement

## Week 4 Success Criteria

By end of Week 4, you should be able to:

1. **Create any dependency** - Functions, classes, with parameters
2. **Build dependency chains** - Complex chains with sub-dependencies
3. **Manage database sessions** - Proper cleanup, resource management
4. **Write middleware** - Request/response processing
5. **Configure CORS** - Secure cross-origin access
6. **Implement authentication** - Via dependencies and middleware
7. **Apply security headers** - Protect against common attacks

## Key Takeaways

- **Dependency Injection enables testing** - Can inject mocks
- **Dependencies cache within request** - Efficient resource usage
- **Yield handles cleanup** - Resources always cleaned up
- **Middleware processes all requests** - Centralized logic
- **CORS prevents attacks** - Specify allowed origins
- **Security headers matter** - Protect against XSS, clickjacking
- **Chains enable complex logic** - Build sophisticated systems

## Common Mistakes to Avoid

1. ❌ Not yielding resources - Leads to resource leaks
2. ❌ Creating new DB connection per query - Use session dependency
3. ❌ Wrong middleware order - Added last runs first!
4. ❌ Allowing "*" origins in CORS - Security risk
5. ❌ Not handling cleanup errors - Use finally block
6. ❌ Not caching dependencies - Recreates unnecessarily
7. ❌ Missing security headers - Exposes to attacks

## Next Steps

After Week 4, you have enterprise-level API patterns. Week 5 introduces databases deeply.

**Before moving to Week 5:**

- [ ] Build Blog API checkpoint project
- [ ] Authentication working completely
- [ ] All dependencies properly chained
- [ ] Database sessions managed correctly
- [ ] Middleware logging requests
- [ ] CORS configured properly
- [ ] Security headers present

---

# ADDITIONAL RESOURCES

## Official Documentation

- FastAPI Dependencies: https://fastapi.tiangolo.com/tutorial/dependencies/
- FastAPI Middleware: https://fastapi.tiangolo.com/advanced/middleware/
- Starlette Middleware: https://www.starlette.io/middleware/

## Database

- SQLAlchemy ORM: https://docs.sqlalchemy.org/
- SQLAlchemy Sessions: https://docs.sqlalchemy.org/orm/session.html

## Security

- OWASP: https://owasp.org/
- Security Headers: https://securityheaders.com/

## Testing

- How to test dependencies: FastAPI testing guide
- Dependency injection testing patterns

---

**End of Week 4 Content**

Excellent! You now have production-ready patterns for authentication, authorization, and resource management. Next week focuses on databases. 🚀