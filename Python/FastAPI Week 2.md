---
title: Fastapi week 2
slug: fastapi-week-2
tags:
  - python
  - fastapi
publish: "true"
style: default
created: 29/07/2026
updated: 31/07/2026
ver: v1
thumbnail: default
description: Detailed fastapi walkthrough for week 2
author_id: "1234"
---

**Duration:** 12-13 hours  
**Learning Style:** Theory-first approach with formal definitions, then code examples  
**Prerequisites:** Week 1 (Decorators, Async/Await, Type Hints)  
**Goal:** Understand FastAPI architecture and build your first complete API

---

## Table of Contents

1. [What is FastAPI? - Formal Introduction](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#what-is-fastapi---formal-introduction)
2. [Your First API - 3 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#your-first-api---3-hours)
3. [Request & Response Basics - 4 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#request--response-basics---4-hours)
4. [Pydantic Models Deep Dive - 3 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#pydantic-models-deep-dive---3-hours)
5. [Dependencies System Introduction - 1.5 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#dependencies-system-introduction---15-hours)
6. [Week 2 Checkpoint Project](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#week-2-checkpoint-project)
7. [Exercise Solutions](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#exercise-solutions)

---

# WHAT IS FASTAPI? - FORMAL INTRODUCTION

## Part 1: Understanding Web Frameworks (0.5 hours)

### Definition: Web Framework

**A web framework is a software toolkit that provides pre-built components, libraries, and patterns to simplify the development of web applications by handling common tasks such as routing, request processing, database interaction, and response generation.**

### Characteristics of a Web Framework:

1. **Request Routing** - Maps HTTP requests to appropriate handler functions
2. **Request Parsing** - Extracts data from URLs, query strings, headers, and request bodies
3. **Response Generation** - Creates formatted HTTP responses
4. **Validation** - Validates incoming data automatically
5. **Error Handling** - Manages exceptions and error responses
6. **Documentation** - Auto-generates API documentation

### Web Framework Categories:

```
Web Frameworks
├── Synchronous (Blocking)
│   ├── Flask (Python) - Lightweight, minimal
│   ├── Django (Python) - Full-featured, batteries included
│   └── Express.js (JavaScript) - Simple, flexible
│
└── Asynchronous (Non-blocking)
    ├── FastAPI (Python) - Modern, fast, automatic validation
    ├── Quart (Python) - Flask but async
    └── Node.js (JavaScript) - Event-driven
```

---

## Part 2: FastAPI's Architecture (1 hour)

### Definition: FastAPI
**FastAPI is a modern, open-source Python web framework built for building fast (high-performance), production-ready REST APIs with automatic data validation, interactive API documentation, and full support for asynchronous programming.**

### Key Components of FastAPI

#### 1. **ASGI (Asynchronous Server Gateway Interface)**

**Definition:** ASGI is a specification for how asynchronous Python web servers communicate with web applications. It's the async successor to WSGI.

```
Comparison: WSGI vs ASGI

WSGI (Synchronous)
┌─────────────┐
│   Client    │
└──────┬──────┘
       │ HTTP Request
       ▼
┌──────-──────────────────────┐
│   WSGI Server (Gunicorn)    │
│  (One thread per request)   │
└──────┬──────────────────────┘
       │ Python Function Call
       ▼
┌─────────────────────────────┐
│   Web Application (Flask)   │
│   (Blocking I/O)            │
└─────────────────────────────┘

ASGI (Asynchronous)
┌─────────────┐
│   Client    │
└──────┬──────┘
       │ HTTP Request
       ▼
┌─────────────────────────────┐
│  ASGI Server (Uvicorn)      │
│  (Event loop, non-blocking) │
└──────┬──────────────────────┘
       │ Async Coroutine
       ▼
┌─────────────────────────────┐
│   Web Application (FastAPI) │
│   (Async/await)             │
└─────────────────────────────┘
```

**Why ASGI matters:**

- Handles multiple concurrent requests efficiently
- While one request waits for I/O (database, network), others can execute
- Better resource utilization
- Can handle thousands of concurrent connections

#### 2. **Starlette (The HTTP Foundation)**

**Definition:** Starlette is a lightweight ASGI framework that provides the low-level HTTP handling. FastAPI is built on top of Starlette.

```
FastAPI Architecture Stack:

┌──────────────────────────┐
│      Your Code           │  (@app.get(), decorators)
│   (Uses FastAPI API)     │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│   FastAPI                │  (Routing, parameter extraction)
│   (High-level API)       │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│   Starlette              │  (HTTP, middleware, background tasks)
│   (Low-level HTTP)       │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│   ASGI Application       │  (Protocol handler)
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│   Uvicorn                │  (ASGI server)
│   (Process Manager)      │
└──────────────────────────┘
```

#### 3. **Pydantic (Data Validation)**

**Definition:** Pydantic is a Python library that uses type hints for data validation and automatic type coercion. It validates data at runtime and raises helpful errors.

**Key benefit for FastAPI:** Combines with type hints to provide automatic validation without writing validation code.

#### 4. **Type Hints (API Documentation)**

**Definition:** Type hints are Python annotations that specify expected data types. FastAPI uses these to auto-generate API documentation and validate requests.

---

## Part 3: FastAPI's Advantages Over Other Frameworks (0.5 hours)

### FastAPI vs Flask

```python
# FLASK (Traditional, Synchronous)
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/users/<int:user_id>', methods=['GET'])
def get_user(user_id):
    # Must manually extract and validate data
    name = request.args.get('name', type=str)
    age = request.args.get('age', type=int)
    
    # Must manually validate
    if age and age < 0:
        return jsonify({"error": "Invalid age"}), 400
    
    # Must manually create response
    return jsonify({"user_id": user_id, "name": name, "age": age})

# Issues:
# - Manual validation needed
# - Synchronous (blocking) by default
# - No automatic documentation
# - Type hints ignored
```

```python
# FASTAPI (Modern, Asynchronous)
from fastapi import FastAPI

app = FastAPI()

@app.get("/users/{user_id}")
async def get_user(user_id: int, name: str, age: int = None):
    # Validation automatic (type hints!)
    # Invalid types rejected with helpful error
    # Synchronous to async handling automatic
    # Documentation auto-generated!
    return {"user_id": user_id, "name": name, "age": age}

# Advantages:
# - Automatic validation via type hints
# - Asynchronous by default
# - OpenAPI docs auto-generated
# - Cleaner code
# - Better performance
```

### FastAPI vs Django

```
             Django              FastAPI
─────────────────────────────────────────────
Setup        Heavy, ORM included  Lightweight
Speed        Medium              Very fast
Learning     Steep               Gentle
Async        Recent addition     Built-in
Auto-docs    Manual              Automatic
Validation   Manual              Automatic
Best for     Full-featured apps  APIs & microservices
```

---

## Part 4: Why FastAPI Is Perfect for Learning (0.5 hours)

### Three Reasons FastAPI Is Ideal for Your Learning Path

**1. Forces Good Practices**

- Type hints are required (you learned this in Week 1!)
- Async/await is default (you learned this in Week 1!)
- Decorators heavily used (you learned this in Week 1!)

**2. Immediate Feedback**

- Interactive API docs (Swagger UI) let you test endpoints instantly
- Validation errors are clear and helpful
- You see results immediately

**3. Production-Ready from the Start**

- Learning FastAPI teaches you production patterns
- Not a "toy" framework to replace later
- Same patterns used by Netflix, Uber, Microsoft

---

# YOUR FIRST API - 3 HOURS

## Part 1: Installation and Setup (0.5 hours)

### Step 1: Create a Project Directory

```bash
# Create project directory
mkdir my_first_fastapi_app
cd my_first_fastapi_app 

# Create virtual environment
python -m venv venv

# Activate virtual environment
# On Windows:
venv\Scripts\activate
# On macOS/Linux:
source venv/bin/activate
```

### Step 2: Install FastAPI and Uvicorn

```bash
# Install FastAPI
pip install fastapi

# Install Uvicorn (ASGI server)
pip install uvicorn

# Verify installation
python -c "import fastapi; print(fastapi.__version__)"
```

### What You Just Installed

| Package     | Purpose       | Role                                |
| ----------- | ------------- | ----------------------------------- |
| **FastAPI** | Web framework | Decorators, routing, validation     |
| **Uvicorn** | ASGI server   | Runs your application, handles HTTP |

### Project Structure

```
my_first_fastapi_app/
├── venv/                    # Virtual environment
├── main.py                  # Your FastAPI application
└── requirements.txt         # Project dependencies
```

Create `requirements.txt`:

```
fastapi==0.104.1
uvicorn==0.24.0
```

Then install from requirements:

```bash
pip install -r requirements.txt
```

---

## Part 2: Understanding the FastAPI Application Object (1 hour)

### Definition: FastAPI Application Instance

A FastAPI application instance is the main Python object that represents your entire API. It's created once at startup and is responsible for routing requests, validation, and response generation.

```python
from fastapi import FastAPI

# This creates a FastAPI application instance
app = FastAPI()

# app is like the "controller" of your API
# All routes are registered to this single app object
```

### Understanding `app = FastAPI()`

```python
from fastapi import FastAPI

# FastAPI() creates an instance
# This instance has methods like:
# - app.get()      -> Handle GET requests
# - app.post()     -> Handle POST requests
# - app.put()      -> Handle PUT requests
# - app.delete()   -> Handle DELETE requests
# - app.patch()    -> Handle PATCH requests

# These methods are DECORATORS (remember Week 1?)
# They decorate your function to make it a route handler

app = FastAPI()

# Optional parameters:
app = FastAPI(
    title="My API",                    # API title (shown in docs)
    description="My first API",        # Description
    version="1.0.0",                   # API version
)
```

### Understanding Path Operations (Routes)

**Definition: A path operation is a mapping between an HTTP method (GET, POST, etc.) and a URL path to a Python function that handles requests to that endpoint.**

```
HTTP Terminology:

┌──────────────────────┐
│   HTTP Request       │
├──────────────────────┤
│ Method: GET          │  <- HTTP Method (verb)
│ Path: /users/123     │  <- URL Path
│ Headers: {...}       │  <- Request metadata
│ Body: {...}          │  <- Request data
└──────────────────────┘
         │
         │ FastAPI Routing
         ▼
┌──────────────────────┐
│ Find Matching Route  │
│ Method: GET          │
│ Path: /users/{id}    │  <- Matches!
└──────────────────────┘
         │
         ▼
┌──────────────────────┐
│ Call Handler Function│
│ async def get_user() │
└──────────────────────┘
         │
         ▼
┌──────────────────────┐
│   HTTP Response      │
├──────────────────────┤
│ Status: 200 OK       │
│ Headers: {...}       │
│ Body: {...}          │
└──────────────────────┘
```

### The Decorator Syntax (Recap from Week 1)

```python
from fastapi import FastAPI

app = FastAPI()

# This decorator syntax:
@app.get("/hello")
def hello():
    return {"message": "Hello"} 

# Is equivalent to:
def hello():
    return {"message": "Hello"}
hello = app.get("/hello")(hello)

# So your function becomes a "route handler"
# that FastAPI calls when it receives a GET request to /hello
```

---

## Part 3: Your First Endpoint (1 hour)

### The Simplest Possible Endpoint

**Definition: An endpoint is a specific URL pattern that accepts requests. For example, `/users/123` is an endpoint that gets user with ID 123.**

```python
from fastapi import FastAPI

# Create app instance
app = FastAPI()

# Define endpoint
@app.get("/hello")
def hello():
    """
    A simple GET endpoint that returns a greeting
    
    When a client makes a GET request to /hello,
    this function is called and its return value
    is sent back as JSON
    """
    return {"message": "Hello, World!"}

# How it works:
# 1. Client makes GET request to /hello
# 2. FastAPI routes to this function
# 3. Function returns dict
# 4. FastAPI converts dict to JSON
# 5. Sends JSON response to client
```

### Running Your API

```bash
# Run the development server
uvicorn main:app --reload

# Output:
# INFO:     Uvicorn running on http://127.0.0.1:8000
# INFO:     Application startup complete
```

**What `uvicorn main:app --reload` means:**

- `main` - Python file name (main.py)
- `app` - Variable name in that file (our FastAPI instance)
- `--reload` - Restart server when you save changes (development only)

### Testing Your Endpoint

#### Option 1: Browser

Navigate to: `http://localhost:8000/hello`

Response:

```json
{"message": "Hello, World!"}
```

#### Option 2: curl (Command line)

```bash
curl http://localhost:8000/hello
```

#### Option 3: Python requests

```python
import requests

response = requests.get("http://localhost:8000/hello")
print(response.json())  # {"message": "Hello, World!"}
```

---

## Part 4: Interactive API Documentation (0.5 hours)

### Automatic Swagger UI

FastAPI automatically generates interactive API documentation!

Navigate to: `http://localhost:8000/docs`

You'll see:

- List of all endpoints
- HTTP method and path
- Button to execute requests live!
- Response preview

### What Makes This Possible

```python
from fastapi import FastAPI

app = FastAPI()

# FastAPI uses the function signature to generate docs
@app.get("/hello")
def hello():
    # This docstring appears in the docs!
    """This is a greeting endpoint"""
    return {"message": "Hello"}

# The @app.get decorator + function info = documentation
# That's why type hints are CRITICAL in FastAPI!
```

## Changing the path to /documentation 
`app = FastAPI(docs_url="/documentation")`
Disabling the automatic docs completely (recommended for production) 
`app = FastAPI(docs_url=None, redoc_url=None)`
### ReDoc (Alternative Documentation)

Navigate to: `http://localhost:8000/redoc`

Same information, different format. Some prefer this layout.

### Disabling Automatic Documentation

```python
# If you don't want the automatic docs (rare):
app = FastAPI(
    docs_url=None,      # Disable Swagger UI
    redoc_url=None      # Disable ReDoc
)
```

---

## Part 5: Understanding HTTP Methods (0.5 hours)

### Definition: HTTP Method

**An HTTP method (or verb) indicates the action the client wants to perform on a resource. The main methods are GET, POST, PUT, DELETE, and PATCH.**

### HTTP Methods and Their Meanings

```
┌─────────┬──────────────────┬──────────────────┐
│ Method  │ Meaning          │ Example          │
├─────────┼──────────────────┼──────────────────┤
│ GET     │ Retrieve data    │ GET /users/123   │
│         │ (Read-only)      │ Gets user 123    │
│         │                  │                  │
│ POST    │ Create new data  │ POST /users      │
│         │ (Send data)      │ Creates new user │
│         │                  │                  │
│ PUT     │ Replace data     │ PUT /users/123   │
│         │ (Update entirely)│ Replaces user123 │
│         │                  │                  │
│ DELETE  │ Remove data      │ DELETE /users/123│
│         │ (Delete)         │ Deletes user 123 │
│         │                  │                  │
│ PATCH   │ Partial update   │ PATCH /users/123 │
│         │ (Update parts)   │ Updates some     │
│         │                  │ fields of user   │
└─────────┴──────────────────┴──────────────────┘

CRUD Mapping:
Create  -> POST
Read    -> GET
Update  -> PUT / PATCH
Delete  -> DELETE
```

### Implementing Different Methods

```python
from fastapi import FastAPI

app = FastAPI()

# GET - Retrieve resource
@app.get("/users/{user_id}")
def get_user(user_id: int):
    return {"user_id": user_id, "name": "Alice"}

# POST - Create resource
@app.post("/users")
def create_user(name: str):
    return {"id": 1, "name": name, "created": True}

# PUT - Replace resource
@app.put("/users/{user_id}")
def update_user(user_id: int, name: str):
    return {"user_id": user_id, "name": name, "updated": True}

# DELETE - Remove resource
@app.delete("/users/{user_id}")
def delete_user(user_id: int):
    return {"deleted": True, "user_id": user_id}

# PATCH - Partial update
@app.patch("/users/{user_id}")
def patch_user(user_id: int, name: str = None, email: str = None):
    return {"user_id": user_id, "updated_fields": ["name", "email"]}
```

---

# REQUEST & RESPONSE BASICS - 4 HOURS

## Part 1: Understanding Request Data Sources (1 hour)

### Definition: Request Data

**Request data is information sent by the client to the server within an HTTP request. It can come from multiple sources: URL path, query string, headers, and request body.**

```
HTTP Request Structure:


Host: api.example.com
Authorization: Bearer token123
Content-Type: application/json


GET /users/123?sort=name&limit=10 HTTP/1.1
│           │        │               │
│           │        │               │           
│           │        │               │
│           │        │               └─ Headers
│           │        │
│           │        └─ Query Parameters
│           │
│           └─ Path Parameters
│
└─ HTTP Method

{"name": "Alice", "age": 30}
	└─ Request Body (JSON)

FastAPI extracts data from ALL these sources:
- Path Parameters: /users/123 -> user_id=123
- Query Parameters: ?sort=name -> sort="name"
- Headers: Authorization -> token="token123"
- Body: JSON payload -> name="Alice", age=30
```

---

## Part 2: Path Parameters (1 hour)

### Definition: Path Parameter

**A path parameter is a variable part of the URL path that identifies a specific resource. For example, in `/users/123`, the `123` is a path parameter identifying a specific user.**

### Syntax and Behavior

```python
from fastapi import FastAPI

app = FastAPI()

# Basic path parameter
@app.get("/users/{user_id}")
def get_user(user_id):
    # user_id is extracted from the URL path
    return {"user_id": user_id, "name": "Alice"}

# When client requests: GET /users/123
# FastAPI extracts: user_id = "123" (as string!)
```

### Type Validation with Path Parameters

**Critical Concept:** Type hints automatically validate path parameters!

```python
from fastapi import FastAPI

app = FastAPI()

# With type hint
@app.get("/users/{user_id}")
def get_user(user_id: int):  # ← Type hint!
    # user_id is automatically converted to int
    # If client sends /users/abc, FastAPI rejects it!
    return {"user_id": user_id, "type": type(user_id).__name__}

# Test cases:
# GET /users/123        → Returns user_id: 123, type: "int" ✓
# GET /users/abc        → Returns 422 Unprocessable Entity ✗
# GET /users/12.5       → Returns 422 Unprocessable Entity ✗

# The error response:
# {
#   "detail": [
#     {
#       "loc": ["path", "user_id"],
#       "msg": "value is not a valid integer",
#       "type": "type_error.integer"
#     }
#   ]
# }
```

### Multiple Path Parameters

```python
from fastapi import FastAPI

app = FastAPI()

# Multiple parameters
@app.get("/users/{user_id}/posts/{post_id}")
def get_post(user_id: int, post_id: int):
    return {
        "user_id": user_id,
        "post_id": post_id,
        "content": "Example post"
    }

# FastAPI matches URL pattern and extracts BOTH
# GET /users/123/posts/456
# → user_id = 123, post_id = 456
```

### Path Parameter Validation with Regex

```python
from fastapi import FastAPI, Path

app = FastAPI()

# Validate format with regex
@app.get("/items/{item_id}")
def get_item(
    item_id: str = Path(..., regex="^[a-z]+$")
    # regex="^[a-z]+$" means: only lowercase letters
):
    return {"item_id": item_id}

# GET /items/abc        → OK ✓
# GET /items/ABC        → 422 Error ✗
# GET /items/abc123     → 422 Error ✗

# The Path() function comes from fastapi
# We'll explain it more fully later
```

---

## Part 3: Query Parameters (1 hour)

### Definition: Query Parameter

**A query parameter is a key-value pair appended to the URL after a `?` symbol. Multiple query parameters are separated by `&`. They're optional by default and used for filtering, sorting, and pagination.**

```
URL Structure:

https://api.example.com/users?age=30&name=alice&sort=created

                             │   │      │   │       │
                             │   │      │   │       └─ Value
                             │   │      │   │
                             │   │      │   └─ Parameter name
                             │   │      │
                             │   │      └─ Separator (&)
                             │   │
                             │   └─ Value
                             │
                             └─ Query string starts with ?
```

### Query Parameters in FastAPI

```python
from fastapi import FastAPI

app = FastAPI()

# Query parameters are function arguments NOT in the path
@app.get("/users")
def get_users(
    skip: int = 0,      # Skip first N items (default 0)
    limit: int = 10     # Return at most N items (default 10)
):
    return {
        "skip": skip,
        "limit": limit,
        "message": f"Returning {limit} items starting from {skip}"
    }

# FastAPI automatically treats these as query parameters
# because they're NOT in the path {user_id}

# Test cases:
# GET /users              → skip=0, limit=10
# GET /users?skip=5       → skip=5, limit=10
# GET /users?limit=20     → skip=0, limit=20
# GET /users?skip=5&limit=20 → skip=5, limit=20
```

### Optional vs Required Query Parameters

```python
from fastapi import FastAPI
from typing import Optional

app = FastAPI()

# Optional: has a default value
@app.get("/items")
def get_items(
    q: Optional[str] = None  # Optional, default None
):
    if q:
        return {"search": q}
    return {"message": "No search query"}

# GET /items              → No search query
# GET /items?q=book       → search: "book"

# Required: no default value
@app.get("/items")
def get_items(
    q: str  # Required! No default
):
    return {"search": q}

# GET /items              → 422 Error! q is required
# GET /items?q=book       → search: "book" ✓
```

### Type Validation for Query Parameters

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/users")
def get_users(
    skip: int = 0,      # Must be integer
    limit: int = 10,    # Must be integer
    active: bool = True # Must be boolean
):
    return {
        "skip": skip,
        "limit": limit,
        "active": active
    }

# Type validation examples:
# GET /users?skip=5&limit=20&active=true
# → skip=5 (int), limit=20 (int), active=True (bool) ✓

# GET /users?skip=abc
# → 422 Error: value is not a valid integer ✗

# GET /users?active=yes
# → 422 Error: value is not a valid boolean ✗
# (Valid bool values: true, false, 1, 0, on, off)
```

### Multiple Query Parameters

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/search")
def search(
    q: str,                    # Search query (required)
    sort: str = "relevance",   # Sort by (optional)
    page: int = 1,             # Page number (optional)
    per_page: int = 10         # Items per page (optional)
):
    return {
        "q": q,
        "sort": sort,
        "page": page,
        "per_page": per_page
    }

# GET /search?q=python&sort=date&page=2&per_page=20
# → Returns all four parameters
```

---

## Part 4: Request Body (1 hour)

### Definition: Request Body

**A request body is the data sent in the body of an HTTP request (typically POST, PUT, PATCH). It's usually formatted as JSON and contains the main data being sent to the server.**

```
HTTP Request with Body:

POST /users HTTP/1.1
Content-Type: application/json

{
  "name": "Alice",
  "email": "alice@example.com",
  "age": 30
}

│                │
│                └─ Request Body (JSON)
│
└─ Request Headers
```

### Receiving JSON Body

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

# Define the shape of data we expect
class User(BaseModel):
    name: str
    email: str
    age: int

# POST endpoint that receives a User in the body
@app.post("/users")
def create_user(user: User):
    # FastAPI automatically:
    # 1. Parses JSON from request body
    # 2. Validates against User model
    # 3. Provides user as Python object
    return {
        "message": f"User {user.name} created",
        "user": user
    }

# When client sends:
# POST /users
# {"name": "Alice", "email": "alice@example.com", "age": 30}
#
# FastAPI does:
# 1. Reads JSON body
# 2. Creates User(name="Alice", email="alice@example.com", age=30)
# 3. Passes to create_user()
# 4. Returns response
```

### Request Body Type Validation

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class User(BaseModel):
    name: str      # Must be string
    email: str     # Must be string
    age: int       # Must be integer

@app.post("/users")
def create_user(user: User):
    return user

# Valid request:
# POST /users
# {"name": "Alice", "email": "alice@example.com", "age": 30}
# → Success ✓

# Invalid: age is string, not int
# POST /users
# {"name": "Alice", "email": "alice@example.com", "age": "thirty"}
# → 422 Error ✗

# Invalid: missing required field
# POST /users
# {"name": "Alice", "email": "alice@example.com"}
# → 422 Error (age is required) ✗

# Invalid: extra fields ignored by default
# POST /users
# {"name": "Alice", "email": "alice@example.com", "age": 30, "phone": "123"}
# → Success ✓ (phone field ignored)
```

---

## Part 5: Response Models and Status Codes (1 hour)

### Definition: Response Model

**A response model is a Pydantic model that defines the structure of the data returned by an endpoint. It ensures consistent response formats and auto-generates documentation.**

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class User(BaseModel):
    id: int
    name: str
    email: str

# Define what response looks like
@app.get("/users/{user_id}", response_model=User)
def get_user(user_id: int):
    # Even if we return extra data,
    # FastAPI only includes fields in User model
    return {
        "id": user_id,
        "name": "Alice",
        "email": "alice@example.com",
        "internal_id": "secret123"  # Won't appear in response!
    }

# Response will be:
# {
#   "id": 1,
#   "name": "Alice",
#   "email": "alice@example.com"
# }
```

### Status Codes

**Definition: An HTTP status code is a three-digit number that indicates the outcome of an HTTP request.**

```
Common Status Codes:

2xx - Success
┌─────┬────────────────────────────────┐
│ 200 │ OK - Request succeeded         │
│ 201 │ Created - Resource created     │
│ 204 │ No Content - Success, no body  │
└─────┴────────────────────────────────┘

3xx - Redirection
┌─────┬────────────────────────────────┐
│ 301 │ Moved Permanently              │
│ 302 │ Found (redirect)               │  
│ 304 │ Not Modified                   │
└─────┴────────────────────────────────┘

4xx - Client Error
┌─────┬────────────────────────────────┐
│ 400 │ Bad Request - Invalid data     │
│ 401 │ Unauthorized - Need auth       │
│ 403 │ Forbidden - Auth but denied    │
│ 404 │ Not Found - Resource missing   │
│ 422 │ Unprocessable Entity - Validation error │
└─────┴────────────────────────────────┘

5xx - Server Error
┌─────┬────────────────────────────────┐
│ 500 │ Internal Server Error          │
│ 503 │ Service Unavailable            │
└─────┴────────────────────────────────┘
```

### Setting Status Codes in FastAPI

```python
from fastapi import FastAPI, status

app = FastAPI()

# Explicit status code for creation
@app.post("/users", status_code=status.HTTP_201_CREATED)
def create_user(name: str):
    return {"id": 1, "name": name}

# When client makes POST request:
# Response status: 201 Created (not default 200)

# Using status constants (cleaner than magic numbers)
@app.post("/users", status_code=status.HTTP_201_CREATED)
def create():
    pass

# Instead of:
@app.post("/users", status_code=201)
def create():
    pass

# Status code per request method defaults:
@app.get("/items")              # Default: 200
@app.post("/items")             # Default: 200 (or 201 for creation)
@app.put("/items/{id}")         # Default: 200
@app.delete("/items/{id}")      # Default: 200
@app.patch("/items/{id}")       # Default: 200
```

---

# PYDANTIC MODELS DEEP DIVE - 3 HOURS

## Part 1: What Are Pydantic Models? (1 hour)

### Definition: Pydantic Model

**A Pydantic model is a Python class that defines the structure, types, and validation rules for data. It uses type hints to automatically validate data at runtime and provide helpful error messages.**

### Why Pydantic Matters

```python
Without Pydantic (Manual Validation):
────────────────────────────────────
def create_user(data):
    # Manual validation - error-prone!
    if "name" not in data:
        return {"error": "name required"}
    if not isinstance(data["name"], str):
        return {"error": "name must be string"}
    if len(data["name"]) < 2:
        return {"error": "name must be >= 2 chars"}
    if "age" not in data:
        return {"error": "age required"}
    if not isinstance(data["age"], int):
        return {"error": "age must be integer"}
    if data["age"] < 0 or data["age"] > 150:
        return {"error": "age must be 0-150"}
    # ... more validation ...
    # Creates mess of validation logic!

With Pydantic (Declarative):
──────────────────────────────
from pydantic import BaseModel

class User(BaseModel):
    name: str           # Type hint
    age: int           # Type hint
    
# DONE! All validation automatic!
# One line of validation code per field instead of many!
```

### Basic Pydantic Model

```python
from pydantic import BaseModel

# Minimal model
class User(BaseModel):
    name: str
    email: str
    age: int

# Usage
user = User(name="Alice", email="alice@example.com", age=30)
print(user.name)     # "Alice"
print(user.age)      # 30

# Access as dict
print(user.dict())   # {"name": "Alice", "email": "alice@example.com", "age": 30}

# Access as JSON
print(user.json())   # '{"name": "Alice", "email": "alice@example.com", "age": 30}'
```

### Automatic Validation

```python
from pydantic import BaseModel
from pydantic import ValidationError

class User(BaseModel):
    name: str
    age: int

# ✓ Valid data
user = User(name="Alice", age=30)  # OK

# ✗ Invalid type
try:
    user = User(name="Alice", age="thirty")
except ValidationError as e:
    print(e)
    # ValidationError: 1 validation error for User
    # age
    #   value is not a valid integer (type=type_error.integer)

# ✗ Missing required field
try:
    user = User(name="Alice")
except ValidationError as e:
    print(e)
    # age: field required

# ✓ Type coercion (automatic conversion)
user = User(name="Alice", age="30")  # String "30" converted to int 30
print(user.age)  # 30 (int type)
```

### Key Pydantic Features

```python
from pydantic import BaseModel, Field
from typing import Optional

class User(BaseModel):
    # Required field
    name: str
    
    # Optional field (can be None)
    email: Optional[str] = None
    
    # Field with default value
    active: bool = True
    
    # Field with constraints
    age: int = Field(gt=0, lt=150)  # Greater than 0, less than 150
    
    # Field with description (shown in API docs)
    bio: Optional[str] = Field(None, max_length=500)

# This model enforces:
# - name is required string
# - email is optional string or None
# - active defaults to True
# - age must be between 1 and 149
# - bio max 500 characters
```

---

## Part 2: Field Validation (0.75 hours)

### Definition: Field Constraint

**A field constraint is a rule that validates the value of a specific field. For example, "age must be between 0 and 150" is a constraint.**

### String Constraints

```python
from pydantic import BaseModel, Field

class Product(BaseModel):
    # Exact length
    sku: str = Field(min_length=5, max_length=5)
    
    # Length range
    name: str = Field(min_length=3, max_length=100)
    
    # Pattern matching (regex)
    email: str = Field(regex="^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$")

# Validation:
# sku="ABC12"     → Valid (length 5)
# sku="AB"        → Invalid (too short)
# name="A"        → Invalid (too short)
# name="Test Product" → Valid
```

### Numeric Constraints

```python
from pydantic import BaseModel, Field

class Person(BaseModel):
    age: int = Field(gt=0, lt=150)          # Greater than 0, less than 150
    height: float = Field(ge=0, le=300)     # Greater or equal, less or equal
    score: int = Field(multiple_of=5)       # Must be multiple of 5

# Constraints:
# gt (greater than)
# ge (greater than or equal)
# lt (less than)
# le (less than or equal)
# multiple_of

# Examples:
# age=25      → Valid
# age=0       → Invalid (not > 0)
# score=10    → Valid (multiple of 5)
# score=11    → Invalid (not multiple of 5)
```

### List Constraints

```python
from pydantic import BaseModel, Field
from typing import List

class Survey(BaseModel):
    # Constrain list length
    answers: List[str] = Field(min_items=1, max_items=10)
    
    # Constrain list items
    tags: List[str] = Field(min_items=0, max_items=5)

# Validation:
# answers=[]  → Invalid (needs at least 1 item)
# answers=["a", "b"]  → Valid
```

---

## Part 3: Nested Models and Relationships (0.75 hours)

### Definition: Nested Model

**A nested model is when one Pydantic model contains another Pydantic model as a field. This allows representing complex, hierarchical data structures.**

```python
from pydantic import BaseModel

# Simple models
class Address(BaseModel):
    street: str
    city: str
    country: str

class User(BaseModel):
    name: str
    email: str
    address: Address  # Nested model!

# Usage
user_data = {
    "name": "Alice",
    "email": "alice@example.com",
    "address": {
        "street": "123 Main St",
        "city": "New York",
        "country": "USA"
    }
}

user = User(**user_data)

# Access nested data
print(user.address.city)  # "New York"
print(user.address)       # Address object
```

### Lists of Nested Models

```python
from pydantic import BaseModel
from typing import List

class Comment(BaseModel):
    text: str
    author: str

class Post(BaseModel):
    title: str
    content: str
    comments: List[Comment]  # List of nested models!

# Usage
post_data = {
    "title": "My Post",
    "content": "Great content",
    "comments": [
        {"text": "Great!", "author": "Bob"},
        {"text": "Thanks!", "author": "Charlie"}
    ]
}

post = Post(**post_data)

# Access
print(len(post.comments))      # 2
print(post.comments[0].author) # "Bob"
```

### Complex Example: Blog Post

```python
from pydantic import BaseModel
from typing import List, Optional
from datetime import datetime

class Author(BaseModel):
    name: str
    email: str

class Comment(BaseModel):
    text: str
    author: Author  # Nested model within nested!
    created_at: datetime

class Post(BaseModel):
    title: str
    content: str
    author: Author
    comments: List[Comment]
    published: bool = False
    created_at: datetime

# This structure represents:
# Post
# ├── Author (title author)
# └── Comment[]
#     ├── text
#     └── Author (comment author)
```

---

# DEPENDENCIES SYSTEM INTRODUCTION - 1.5 HOURS

## Part 1: Understanding Dependency Injection (0.75 hours)

### Definition: Dependency Injection

**Dependency injection is a design pattern where a function receives its required dependencies as parameters rather than creating them internally. This makes code more testable, reusable, and maintainable.**

### Problem Without Dependency Injection

```python
# ❌ Without DI - Hard to test, tightly coupled
class UserDatabase:
    def get_user(self, user_id: int):
        # Actual database call
        return {"id": user_id, "name": "Alice"}

def get_user_endpoint(user_id: int):
    # Creates its own database - can't test with fake DB!
    db = UserDatabase()
    user = db.get_user(user_id)
    return user

# To test this, must test against real database!
```

### Solution With Dependency Injection

```python
# ✅ With DI - Easy to test, loosely coupled
class UserDatabase:
    def get_user(self, user_id: int):
        return {"id": user_id, "name": "Alice"}

def get_user_endpoint(user_id: int, db: UserDatabase):
    # Receives db as parameter!
    user = db.get_user(user_id)
    return user

# Testing with fake database
class FakeDatabase:
    def get_user(self, user_id: int):
        return {"id": user_id, "name": "Test User"}

def test():
    fake_db = FakeDatabase()
    result = get_user_endpoint(1, fake_db)  # Pass fake DB
    assert result["name"] == "Test User"
```

### Dependency Injection in FastAPI

```python
from fastapi import FastAPI, Depends

app = FastAPI()

# Define a dependency (any callable)
def get_database():
    """This function creates and returns a database connection"""
    db = {"users": [{"id": 1, "name": "Alice"}]}
    return db

# Use the dependency with Depends()
@app.get("/users/{user_id}")
def get_user(user_id: int, db = Depends(get_database)):
    # FastAPI automatically:
    # 1. Calls get_database()
    # 2. Passes the result as db parameter
    # 3. Caches result within request scope
    return {"user_id": user_id, "database": db}

# When client requests GET /users/1:
# 1. FastAPI calls get_database() → db connection
# 2. FastAPI calls get_user(1, db=connection)
# 3. Returns response
```

---

## Part 2: FastAPI Dependencies (0.75 hours)

### Using Dependencies

```python
from fastapi import FastAPI, Depends

app = FastAPI()

# Simple dependency function
def get_query(q: str = None):
    """Extract query parameter"""
    if q is None:
        return ""
    return q.lower()

# Use with Depends()
@app.get("/items")
def search_items(query: str = Depends(get_query)):
    # Depends() calls get_query() and passes result
    return {"query": query}

# When client makes GET /items?q=PYTHON:
# 1. FastAPI calls get_query(q="PYTHON")
# 2. Returns "python"
# 3. Passes to search_items(query="python")
```

### Dependency Functions with Parameters

```python
from fastapi import FastAPI, Depends

app = FastAPI()

# Dependency with its own parameters
def get_pagination(skip: int = 0, limit: int = 10):
    """Extract pagination parameters"""
    return {"skip": skip, "limit": limit}

@app.get("/items")
def list_items(pagination: dict = Depends(get_pagination)):
    return {
        "items": [1, 2, 3],
        "pagination": pagination
    }

# GET /items?skip=5&limit=20 returns:
# {
#   "items": [1, 2, 3],
#   "pagination": {"skip": 5, "limit": 20}
# }
```

### Chaining Dependencies

```python
from fastapi import FastAPI, Depends

app = FastAPI()

# Dependency 1
def get_database():
    return {"connection": "db_connection"}

# Dependency 2 depends on Dependency 1!
def get_user_from_db(db: dict = Depends(get_database)):
    return db.get("user_data", {"id": 1, "name": "Alice"})

# Use chained dependency
@app.get("/user")
def get_user(user: dict = Depends(get_user_from_db)):
    return user

# Execution flow:
# 1. get_database() called → returns db
# 2. get_user_from_db(db=db) called → returns user
# 3. get_user(user=user) called → returns user
```

### Why Dependencies Matter

Dependencies enable:

1. **Code Reuse** - Share logic across endpoints
2. **Testability** - Inject fake dependencies for testing
3. **Separation of Concerns** - Extract logic from endpoints
4. **Caching** - Dependencies cached per request
5. **Automatic Documentation** - Dependencies shown in docs

---

## Part 3: Real-World Dependency Examples (0.75 hours)

### Example 1: Logging Dependency

```python
from fastapi import FastAPI, Depends
from datetime import datetime

app = FastAPI()

def log_request(path: str, method: str = "GET"):
    """Dependency that logs requests"""
    timestamp = datetime.now().isoformat()
    print(f"[{timestamp}] {method} {path}")
    return {"path": path, "method": method}

@app.get("/items")
def list_items(log: dict = Depends(log_request)):
    return {"items": [1, 2, 3], "log": log}

# Every request logs automatically via dependency!
```

### Example 2: Authentication Dependency (Preview of Week 8)

```python
from fastapi import FastAPI, Depends, HTTPException, status

app = FastAPI()

def get_current_user(token: str = None):
    """Validate token and return user"""
    if not token or token != "valid_token":
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid token"
        )
    return {"id": 1, "name": "Alice"}

@app.get("/protected")
def protected_route(user: dict = Depends(get_current_user)):
    return {"message": f"Hello, {user['name']}"}

# Without valid token: 401 Unauthorized
# With valid token: Returns protected data
```

### Example 3: Query Parameter Dependency

```python
from fastapi import FastAPI, Depends

app = FastAPI()

class FilterParams:
    def __init__(self, skip: int = 0, limit: int = 10):
        self.skip = skip
        self.limit = limit

@app.get("/items")
def list_items(filters: FilterParams = Depends()):
    return {
        "skip": filters.skip,
        "limit": filters.limit,
        "items": [1, 2, 3]
    }

# GET /items?skip=5&limit=20
# FastAPI creates FilterParams(skip=5, limit=20)
# Much cleaner than multiple parameters!
```

---

# WEEK 2 CHECKPOINT PROJECT

## Project: User Management API

### Objective

Create a complete API that demonstrates all Week 2 concepts:

- Path parameters
- Query parameters
- Request body
- Response models
- Status codes
- Pydantic validation
- Dependencies

### Project Requirements

#### Part 1: Data Models

Create Pydantic models:

```python
from pydantic import BaseModel, Field, EmailStr
from typing import Optional

class User(BaseModel):
    """User data model"""
    id: int
    name: str = Field(..., min_length=2, max_length=100)
    email: str  # Should validate email format
    age: Optional[int] = Field(None, ge=0, le=150)
    is_active: bool = True

class UserCreate(BaseModel):
    """Model for creating new users (no id)"""
    name: str = Field(..., min_length=2, max_length=100)
    email: str
    age: Optional[int] = Field(None, ge=0, le=150)
```

#### Part 2: In-Memory Data Store

```python
# Simulate database with list
users_db = [
    {"id": 1, "name": "Alice", "email": "alice@example.com", "age": 30, "is_active": True},
    {"id": 2, "name": "Bob", "email": "bob@example.com", "age": 25, "is_active": True},
]
```

#### Part 3: Endpoints

Implement these endpoints:

**1. List All Users**

```
GET /users
- Query parameters: skip (int, default 0), limit (int, default 10)
- Returns: List[User]
- Example: GET /users?skip=0&limit=10
```

**2. Get User by ID**

```
GET /users/{user_id}
- Path parameter: user_id (int)
- Returns: User
- Error: 404 if user not found
- Example: GET /users/1
```

**3. Create User**

```
POST /users
- Request body: UserCreate
- Returns: User (with generated id)
- Status: 201 Created
- Validation: Name length 2-100, age 0-150
- Example body: {"name": "Charlie", "email": "charlie@example.com", "age": 28}
```

**4. Update User**

```
PUT /users/{user_id}
- Path parameter: user_id (int)
- Request body: UserCreate
- Returns: User
- Error: 404 if user not found
- Example: PUT /users/1 with body
```

**5. Delete User**

```
DELETE /users/{user_id}
- Path parameter: user_id (int)
- Returns: {"message": "User deleted"}
- Status: 200 OK
- Error: 404 if user not found
```

**6. Search Users**

```
GET /users/search
- Query parameter: q (str, required)
- Returns: List[User] (filtered by name containing q)
- Example: GET /users/search?q=ali
```

#### Part 4: Implementation Guide

```python
from fastapi import FastAPI, HTTPException, status, Depends
from pydantic import BaseModel, Field
from typing import List, Optional

app = FastAPI(title="User Management API", version="1.0.0")

class User(BaseModel):
    id: int
    name: str = Field(..., min_length=2, max_length=100)
    email: str
    age: Optional[int] = Field(None, ge=0, le=150)
    is_active: bool = True

class UserCreate(BaseModel):
    name: str = Field(..., min_length=2, max_length=100)
    email: str
    age: Optional[int] = Field(None, ge=0, le=150)

# Mock database
users_db = [
    {"id": 1, "name": "Alice", "email": "alice@example.com", "age": 30, "is_active": True},
    {"id": 2, "name": "Bob", "email": "bob@example.com", "age": 25, "is_active": True},
]

# Dependency: Get next user ID
def get_next_user_id():
    """Get the next available user ID"""
    if not users_db:
        return 1
    return max(u["id"] for u in users_db) + 1

# TODO: Implement all 6 endpoints here
```

#### Part 5: Testing with curl

```bash
# List users
curl http://localhost:8000/users

# Get user
curl http://localhost:8000/users/1

# Create user
curl -X POST http://localhost:8000/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Charlie", "email": "charlie@example.com", "age": 28}'

# Update user
curl -X PUT http://localhost:8000/users/1 \
  -H "Content-Type: application/json" \
  -d '{"name": "Alice Updated", "email": "alice.new@example.com", "age": 31}'

# Delete user
curl -X DELETE http://localhost:8000/users/1

# Search
curl "http://localhost:8000/users/search?q=alice"
```

### Deliverables

Create `week_2_user_api.py` with:

1. ✓ Pydantic models (User, UserCreate)
2. ✓ All 6 endpoints implemented
3. ✓ Proper status codes
4. ✓ Error handling (404s)
5. ✓ Query parameter validation
6. ✓ Request body validation
7. ✓ Comments explaining each endpoint

### Success Criteria

- [ ] Can list all users with pagination
- [ ] Can get specific user by ID
- [ ] Can create user with validation
- [ ] Can update user
- [ ] Can delete user
- [ ] Can search users
- [ ] All endpoints have proper status codes
- [ ] API docs work in Swagger UI
- [ ] Tests pass with curl/Postman

---

# EXERCISE SOLUTIONS

## Exercise 1: Basic Endpoints

**Problem:** Create endpoints for a simple task manager

**Solution:**

```python
from fastapi import FastAPI, HTTPException, status
from pydantic import BaseModel
from typing import List, Optional

app = FastAPI()

class Task(BaseModel):
    id: int
    title: str
    description: Optional[str] = None
    completed: bool = False

tasks_db = [
    {"id": 1, "title": "Learn FastAPI", "description": "Complete the course", "completed": False},
    {"id": 2, "title": "Build API", "description": "Create user management API", "completed": False},
]

@app.get("/tasks", response_model=List[Task])
def list_tasks():
    return tasks_db

@app.get("/tasks/{task_id}", response_model=Task)
def get_task(task_id: int):
    task = next((t for t in tasks_db if t["id"] == task_id), None)
    if not task:
        raise HTTPException(status_code=404, detail="Task not found")
    return task

@app.post("/tasks", response_model=Task, status_code=status.HTTP_201_CREATED)
def create_task(task: Task):
    task_dict = task.dict()
    task_dict["id"] = max(t["id"] for t in tasks_db) + 1
    tasks_db.append(task_dict)
    return task_dict
```

## Exercise 2: Query Parameter Filtering

**Problem:** Add filtering to list endpoint

**Solution:**

```python
@app.get("/tasks", response_model=List[Task])
def list_tasks(completed: Optional[bool] = None, skip: int = 0, limit: int = 10):
    filtered = tasks_db
    
    if completed is not None:
        filtered = [t for t in filtered if t["completed"] == completed]
    
    return filtered[skip:skip+limit]

# Usage:
# GET /tasks                    -> All tasks
# GET /tasks?completed=true     -> Only completed
# GET /tasks?completed=false    -> Only incomplete
# GET /tasks?skip=5&limit=10    -> Paginated results
```

## Exercise 3: Request Body Validation

**Problem:** Create endpoint that validates user input strictly

**Solution:**

```python
from pydantic import BaseModel, Field, validator

class StrictTask(BaseModel):
    title: str = Field(..., min_length=3, max_length=200)
    description: str = Field("", max_length=1000)
    priority: int = Field(1, ge=1, le=5)
    
    @validator('title')
    def title_must_not_be_all_caps(cls, v):
        if v.isupper():
            raise ValueError('Title cannot be all caps')
        return v

@app.post("/tasks/strict", status_code=status.HTTP_201_CREATED)
def create_strict_task(task: StrictTask):
    # All validation automatic!
    return {"created": True, "task": task}

# This will reject:
# - title too short or too long
# - description too long
# - priority outside 1-5
# - title in ALL CAPS
```

---

# SUMMARY AND CHECKLIST

## Week 2 Learning Path Summary

**Total Time: 12-13 Hours**

### What is FastAPI (1.5 hours)

- ✅ Understand ASGI vs WSGI
- ✅ Know Starlette foundation
- ✅ Understand Pydantic role
- ✅ Know FastAPI advantages
- ✅ Understand request/response cycle

### Your First API (3 hours)

- ✅ Setup project and install FastAPI
- ✅ Create FastAPI application
- ✅ Understand path operations
- ✅ Create first endpoint
- ✅ Test with browser and curl
- ✅ Use Swagger UI docs
- ✅ Understand HTTP methods (GET, POST, PUT, DELETE, PATCH)

### Request & Response (4 hours)

- ✅ Extract path parameters
- ✅ Extract query parameters
- ✅ Receive request body JSON
- ✅ Type validation automatic
- ✅ Define response models
- ✅ Set status codes
- ✅ Handle errors (404, 422)

### Pydantic Models (3 hours)

- ✅ Create basic models
- ✅ Understand automatic validation
- ✅ Use field constraints (min_length, gt, etc.)
- ✅ Create nested models
- ✅ Use lists in models
- ✅ Optional vs required fields

### Dependencies (1.5 hours)

- ✅ Understand dependency injection pattern
- ✅ Use Depends() to inject dependencies
- ✅ Create dependency functions
- ✅ Chain dependencies
- ✅ Understand dependency scope

## Week 2 Success Criteria

By end of Week 2, you should be able to:

1. **Create any FastAPI endpoint** - GET, POST, PUT, DELETE
2. **Extract data from all sources** - Path, query, body, headers
3. **Validate data automatically** - Using Pydantic models
4. **Define response structures** - Using response_model
5. **Use dependencies** - Share logic across endpoints
6. **Understand request flow** - Client → FastAPI → Handler → Response
7. **Use Swagger UI** - Test APIs without external tools
8. **Handle errors** - Return proper status codes and error responses

## Progression Indicators

|After This...|You Can...|
|---|---|
|1.5 hrs|Explain FastAPI architecture|
|3 hrs|Build working endpoints|
|7 hrs|Validate complex data structures|
|12 hrs|Create production-like APIs|

## Key Concepts to Review

- **ASGI/WSGI** - Why async matters
- **Type Hints + Pydantic** - Automatic validation
- **Path vs Query Parameters** - Different data sources
- **Status Codes** - Proper HTTP semantics
- **Response Models** - Consistent responses
- **Dependencies** - Code reuse and testability

---

# ADDITIONAL RESOURCES

## Official Documentation

- FastAPI Docs: https://fastapi.tiangolo.com/
- Pydantic Docs: https://docs.pydantic.dev/
- Starlette Docs: https://www.starlette.io/

## Video Resources

- FastAPI Full Tutorial: https://www.youtube.com/watch?v=7t2alSnE2-I
- Pydantic Tutorial: https://www.youtube.com/watch?v=DfxtWeSVzc0

## Practice Exercises

- Create CRUD API for your favorite data model
- Build API for: Books, Products, Tweets, Notes
- Add search/filter to list endpoints
- Implement pagination

---

**End of Week 2 Content**

Congratulations on completing Week 1 and Week 2! You now have the fundamentals of FastAPI. Week 3 starts advanced routing and validation. 🚀

Remember:

- FastAPI = Type hints + Pydantic + Async + Starlette
- Validation = Automatic via Pydantic
- Documentation = Auto-generated from code
- Testing = Use Swagger UI or curl

Good luck! 🎉