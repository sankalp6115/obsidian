**Duration:** 12-13 hours  
**Learning Style:** Formal definitions + Theory + Practical examples  
**Prerequisites:** Week 1 (Python foundations) + Week 2 (FastAPI basics)  
**Goal:** Master advanced request/response handling and comprehensive validation

---
## Table of Contents

1. [Path Operations Deep Dive - 2.5 Hours](#path-operations-deep-dive---25-hours)
2. [Advanced Request Details - 3 Hours](#advanced-request-details---3-hours)
3. [Advanced Response Details - 2.5 Hours](#advanced-response-details---25-hours)
4. [Error Handling & Validation - 2.5 Hours](#error-handling--validation---25-hours)
5. [Documentation & OpenAPI - 1.5 Hours](#documentation--openapi---15-hours)
6. [Week 3 Checkpoint Project](#week-3-checkpoint-project)

---

# PATH OPERATIONS DEEP DIVE - 2.5 HOURS

## Part 1: Path Parameter Validation and Types

### Formal Definition: Path Parameter

**Path Parameter:** A variable component of the URL path that identifies or filters a specific resource. Path parameters are:

- Required (must be provided in the URL)
- Part of the URL path itself (not query string)
- Used to specify which resource to operate on

**Distinction:** `/users/123` → path parameter is `123`  
**NOT:** `/users?id=123` → this is a query parameter

### Understanding Path Parameter Types

**Type System in FastAPI:**

```python
# FastAPI automatically converts path parameters to specified type
# Based on the type hint in function parameter

@app.get("/items/{item_id}")
def read_item(item_id: int):
    # item_id is automatically converted from string to int
    # URL: /items/123 → item_id = 123 (integer)
    # URL: /items/abc → FastAPI returns 422 validation error
    return {"item_id": item_id}
```

**Supported Path Parameter Types:**

```python
from fastapi import FastAPI

app = FastAPI()

# 1. Integer path parameter
@app.get("/users/{user_id}")
def get_user(user_id: int):
    """
    Converts "123" from URL to integer 123
    Validation: Must be valid integer
    """
    return {"user_id": user_id, "type": "int"}

# 2. String path parameter (default if no type specified)
@app.get("/items/{item_name}")
def get_item_by_name(item_name: str):
    """
    Keeps as string
    URL: /items/widget → item_name = "widget"
    """
    return {"item_name": item_name}

# 3. Float path parameter
@app.get("/prices/{price}")
def get_price(price: float):
    """
    Converts to float
    URL: /prices/9.99 → price = 9.99
    """
    return {"price": price}

# 4. Boolean path parameter
@app.get("/status/{is_active}")
def get_status(is_active: bool):
    """
    Converts to boolean
    Accepts: true, True, TRUE, 1
    Accepts: false, False, FALSE, 0
    URL: /status/true → is_active = True
    """
    return {"is_active": is_active}

# 5. UUID path parameter (unique identifiers)
from uuid import UUID

@app.get("/documents/{document_id}")
def get_document(document_id: UUID):
    """
    Validates UUID format
    URL: /documents/550e8400-e29b-41d4-a716-446655440000
    """
    return {"document_id": document_id}
```

### Path Parameter Validation with Constraints

**Formal Definition: Constraint** A rule that restricts the valid values for a parameter. Constraints are checked before the endpoint function is called.

```python
from fastapi import FastAPI, Path
from typing import Optional

app = FastAPI()

# Constraint: Minimum value
@app.get("/items/{item_id}")
def read_item(item_id: int = Path(..., gt=0)):
    """
    Path(...) tells FastAPI this is a path parameter with constraints
    gt=0 means "greater than 0"
    
    URL: /items/123 → Valid
    URL: /items/0 → 422 Validation Error
    URL: /items/-5 → 422 Validation Error
    """
    return {"item_id": item_id}

# Common numeric constraints:
@app.get("/range/{number}")
def check_range(
    number: int = Path(..., gt=0, lt=100)
):
    """
    gt=0: greater than 0
    lt=100: less than 100
    So: 0 < number < 100
    
    Valid: 1-99
    Invalid: 0, 100, 150, -5
    """
    return {"number": number}

# Using gte and lte (inclusive)
@app.get("/score/{score}")
def check_score(
    score: int = Path(..., gte=0, lte=100)
):
    """
    gte=0: greater than or equal to 0
    lte=100: less than or equal to 100
    So: 0 <= score <= 100
    
    Valid: 0-100 (inclusive)
    Invalid: -1, 101
    """
    return {"score": score}

# Constraints for floats
@app.get("/percentage/{value}")
def check_percentage(
    value: float = Path(..., ge=0.0, le=100.0)
):
    """
    ge: greater than or equal to
    le: less than or equal to
    Works the same for floats
    """
    return {"percentage": value}
```

### String Path Parameter Validation

```python
from fastapi import Path

app = FastAPI()

# String length validation
@app.get("/search/{query}")
def search(
    query: str = Path(..., min_length=3, max_length=50)
):
    """
    min_length=3: Must be at least 3 characters
    max_length=50: Cannot exceed 50 characters
    
    Valid: "cat", "hello world"
    Invalid: "ab" (too short), "a"*60 (too long)
    """
    return {"query": query}

# Regular expression validation
@app.get("/code/{code}")
def validate_code(
    code: str = Path(..., regex="^[A-Z]{3}[0-9]{3}$")
):
    """
    regex="^[A-Z]{3}[0-9]{3}$" means:
    - ^ start of string
    - [A-Z]{3} exactly 3 uppercase letters
    - [0-9]{3} exactly 3 digits
    - $ end of string
    
    Valid: ABC123, XYZ999
    Invalid: abc123 (lowercase), AB12 (wrong format)
    """
    return {"code": code}

# Alphanumeric codes
@app.get("/product/{sku}")
def get_product(
    sku: str = Path(..., regex="^[A-Z0-9]+$")
):
    """
    ^[A-Z0-9]+$ means:
    Only uppercase letters and digits, at least 1 character
    
    Valid: SKU123, PROD
    Invalid: sku123 (lowercase), SKU-123 (contains hyphen)
    """
    return {"sku": sku}
```

### Enum Path Parameters

**Formal Definition: Enum (Enumeration)** A type that restricts a parameter to a fixed set of predefined values. Only these exact values are allowed.

```python
from fastapi import FastAPI, Path
from enum import Enum

app = FastAPI()

# Define an Enum
class ItemType(str, Enum):
    """
    Inherits from both str and Enum
    Allows FastAPI to serialize to JSON as string
    """
    BOOK = "book"
    MOVIE = "movie"
    MUSIC = "music"
    OTHER = "other"

@app.get("/items/{item_type}")
def read_items_by_type(item_type: ItemType):
    """
    item_type must be one of: book, movie, music, other
    
    Valid URLs:
    - /items/book
    - /items/movie
    
    Invalid URLs:
    - /items/video (not in enum) → 422 error
    - /items/BOOK (case sensitive) → 422 error
    """
    if item_type == ItemType.BOOK:
        return {"items": ["Book 1", "Book 2"]}
    elif item_type == ItemType.MOVIE:
        return {"items": ["Movie 1", "Movie 2"]}
    return {"items": []}

# Numeric Enum
class Priority(int, Enum):
    """Priority levels as integers"""
    LOW = 1
    MEDIUM = 2
    HIGH = 3

@app.get("/tasks/{priority}")
def get_tasks(priority: Priority):
    """
    Valid: /tasks/1, /tasks/2, /tasks/3
    Invalid: /tasks/4, /tasks/5
    """
    return {"priority": priority}

# Get enum value in response
@app.get("/status/{status_enum}")
def check_status(status_enum: ItemType):
    """
    FastAPI documentation shows which values are accepted
    Interactive docs show a dropdown menu!
    """
    return {
        "selected": status_enum,
        "value": status_enum.value,  # "book", "movie", etc.
        "name": status_enum.name     # "BOOK", "MOVIE", etc.
    }
```

### Multiple Path Parameters with Validation

```python
from fastapi import Path
from datetime import date

app = FastAPI()

# Multiple path parameters with constraints
@app.get("/users/{user_id}/posts/{post_id}")
def get_user_post(
    user_id: int = Path(..., gt=0),
    post_id: int = Path(..., gt=0)
):
    """
    Both parameters must be greater than 0
    
    Valid: /users/123/posts/456
    Invalid: /users/0/posts/456
    """
    return {"user_id": user_id, "post_id": post_id}

# Path parameters with descriptions
@app.get("/events/{year}/{month}/{day}")
def get_events(
    year: int = Path(..., ge=2000, le=2100, description="Year (2000-2100)"),
    month: int = Path(..., ge=1, le=12, description="Month (1-12)"),
    day: int = Path(..., ge=1, le=31, description="Day (1-31)")
):
    """
    description parameter shows up in OpenAPI docs
    Helps API users understand constraints
    """
    return {"date": f"{year}-{month:02d}-{day:02d}"}

# Complex: UUID with regex
@app.get("/org/{org_id}/project/{project_code}")
def get_project(
    org_id: int = Path(..., gt=0, description="Organization ID"),
    project_code: str = Path(..., regex="^[A-Z]{3}[0-9]{4}$", 
                              description="Project code like ABC1234")
):
    """Combining different types and constraints"""
    return {"org_id": org_id, "project_code": project_code}
```

---

## Part 2: OpenAPI and Documentation in Paths

**Formal Definition: OpenAPI** An open standard for describing REST APIs. FastAPI automatically generates OpenAPI specs and interactive documentation from your code.

```python
from fastapi import Path

app = FastAPI()

@app.get("/items/{item_id}")
def read_item(
    item_id: int = Path(
        ...,                    # Required
        gt=0,                  # Constraint
        title="Item ID",       # Display name in docs
        description="The ID of the item to retrieve. Must be positive.",
        example=42             # Example value shown in docs
    )
):
    """
    Get a specific item by ID.
    
    This appears in documentation:
    - title: Shows "Item ID" as field name
    - description: Explains the constraint
    - example: Shows "42" as example value
    - In interactive /docs, users see a form with this info
    """
    return {"item_id": item_id}
```

---

# ADVANCED REQUEST DETAILS - 3 HOURS

## Part 1: Multiple Body Parameters

### Formal Definition: Request Body

**Request Body:** The data sent with HTTP requests (typically POST, PUT, PATCH) containing information about the resource being created or modified.

**Key distinction from parameters:**

- Path parameters: Identify WHICH resource (from URL)
- Query parameters: Filter or sort (from URL query string)
- Body parameters: Data FOR that resource (from request body)

### Single Body Parameter (Pydantic Model)

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str
    price: float

@app.post("/items/")
def create_item(item: Item):
    """
    Single Pydantic model as body
    
    FastAPI expects JSON body matching Item structure:
    {
      "name": "Widget",
      "description": "A useful widget",
      "price": 9.99
    }
    """
    return item
```

### Multiple Body Parameters with Body()

**Formal Definition: Body()** A function in FastAPI that explicitly declares how to handle request body parameters. Allows mixing path, query, and body parameters with precise control.

```python
from fastapi import FastAPI, Body
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float

class User(BaseModel):
    username: str
    email: str

# Without Body() - FastAPI doesn't know if 'importance' is body or query
@app.post("/items/")
def create_item_with_importance(
    item: Item,
    importance: int = Body(...)  # Use Body() to make it explicit
):
    """
    Body(...) tells FastAPI:
    - importance comes from request body (not query)
    - It's required (...)
    
    Expected JSON:
    {
      "item": {
        "name": "Widget",
        "price": 9.99
      },
      "importance": 5
    }
    """
    return {"item": item, "importance": importance}

# Multiple models
@app.post("/purchases/")
def create_purchase(
    item: Item = Body(...),
    user: User = Body(...),
    quantity: int = Body(..., gt=0)
):
    """
    Multiple body parameters
    FastAPI embeds each in a separate key
    
    Expected JSON:
    {
      "item": {"name": "Widget", "price": 9.99},
      "user": {"username": "alice", "email": "alice@example.com"},
      "quantity": 2
    }
    """
    return {
        "item": item,
        "user": user,
        "quantity": quantity,
        "total": item.price * quantity
    }
```

### Singular Value in Body

```python
from fastapi import Body

app = FastAPI()

# Single scalar value in body
@app.put("/items/{item_id}")
def update_item_price(
    item_id: int,
    price: float = Body(...)
):
    """
    Without Body(), FastAPI treats price as query parameter
    With Body(...), FastAPI expects it in JSON body
    
    Request:
    PUT /items/1
    Content-Type: application/json
    
    9.99  <- JSON value (plain number)
    
    Note: This looks odd, usually you'd use a Pydantic model
    But it's useful for updating single fields
    """
    return {"item_id": item_id, "new_price": price}

# This is usually avoided in favor of:
class PriceUpdate(BaseModel):
    price: float

@app.put("/items/{item_id}")
def update_item_price_proper(
    item_id: int,
    price_update: PriceUpdate
):
    """
    Better: Use Pydantic model
    Clearer what data is expected
    """
    return {"item_id": item_id, "new_price": price_update.price}
```

### Embedding Models

**Formal Definition: Embedding** Wrapping a Pydantic model in a JSON object with a specific key. Used when you want a more structured request body.

```python
from fastapi import FastAPI, Body
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float

# Without embedding (default)
@app.post("/items/default")
def create_item_default(item: Item):
    """
    Request body is just the item:
    {
      "name": "Widget",
      "price": 9.99
    }
    """
    return item

# With embedding
@app.post("/items/embedded")
def create_item_embedded(item: Item = Body(..., embed=True)):
    """
    embed=True wraps the model in a key
    Request body must be:
    {
      "item": {
        "name": "Widget",
        "price": 9.99
      }
    }
    
    Use when:
    - API expects nested structure
    - Multiple models need clear separation
    - Maintaining backward compatibility
    """
    return item
```

---

## Part 2: Form Data

**Formal Definition: Form Data** Data sent via HTML forms using `application/x-www-form-urlencoded` encoding. Different from JSON body - uses key-value pairs like URL parameters.

**When to use:**

- HTML form submissions (login, registration)
- File uploads with metadata
- Legacy API compatibility

```python
from fastapi import FastAPI, Form

app = FastAPI()

# Simple form data
@app.post("/login/")
def login(
    username: str = Form(...),
    password: str = Form(...)
):
    """
    Expects form data (NOT JSON)
    
    Request:
    POST /login/
    Content-Type: application/x-www-form-urlencoded
    
    username=alice&password=secret123
    
    Note: Use Form() to tell FastAPI these come from form, not JSON
    """
    return {"username": username, "access_token": "fake-token"}

# Form data with validation
@app.post("/register/")
def register(
    username: str = Form(..., min_length=3, max_length=20),
    email: str = Form(..., regex=r"^[\w\.-]+@[\w\.-]+\.\w+$"),
    age: int = Form(..., ge=18, le=120)
):
    """
    Form data with Field constraints
    Same validation as Pydantic, but for form data
    """
    return {"username": username, "email": email, "age": age}

# Mixed: Form + JSON
@app.post("/mixed/")
def mixed_data(
    username: str = Form(...),
    metadata: dict = Body(...)  # JSON body
):
    """
    Can't mix JSON and form in same request!
    This endpoint expects:
    1. Form data with username
    2. JSON body for metadata
    
    This requires special handling in client
    Usually NOT recommended - keep one format
    """
    return {"username": username, "metadata": metadata}
```

---

## Part 3: File Uploads

**Formal Definition: File Upload** Sending binary file data to server via HTTP request. Uses `multipart/form-data` encoding which supports both file binary data and regular form fields.

### Single File Upload

```python
from fastapi import FastAPI, File, UploadFile

app = FastAPI()

# Simple file upload
@app.post("/upload/")
def upload_file(file: UploadFile = File(...)):
    """
    UploadFile is FastAPI's wrapper around uploaded file
    
    Client sends:
    POST /upload/
    Content-Type: multipart/form-data
    
    [binary file data]
    """
    return {
        "filename": file.filename,
        "content_type": file.content_type,
        "size": len(file.file.read())
    }

# Actually read file content
@app.post("/upload-process/")
async def upload_and_process(file: UploadFile = File(...)):
    """
    Read file contents and process
    
    UploadFile provides:
    - filename: Original filename
    - content_type: MIME type (image/png, application/pdf, etc.)
    - file: File-like object for reading bytes
    """
    contents = await file.read()  # Read entire file
    return {
        "filename": file.filename,
        "size_bytes": len(contents),
        "first_100_bytes": contents[:100]
    }
```

### Multiple File Uploads

```python
from fastapi import File, UploadFile
from typing import List

app = FastAPI()

# Multiple files
@app.post("/upload-multiple/")
async def upload_multiple(files: List[UploadFile] = File(...)):
    """
    Upload multiple files at once
    
    Client sends multiple files:
    POST /upload-multiple/
    Content-Type: multipart/form-data
    
    [file1]
    [file2]
    [file3]
    """
    results = []
    for file in files:
        contents = await file.read()
        results.append({
            "filename": file.filename,
            "size": len(contents)
        })
    return results
```

### File Upload with Form Data

```python
from fastapi import Form, File, UploadFile

app = FastAPI()

# File + form fields
@app.post("/upload-with-metadata/")
async def upload_with_metadata(
    file: UploadFile = File(...),
    description: str = Form(...),
    tags: str = Form(...)  # Comma-separated
):
    """
    Upload file with additional form fields
    
    Client sends:
    - file (binary)
    - description (text)
    - tags (text)
    
    All in multipart/form-data format
    """
    contents = await file.read()
    return {
        "filename": file.filename,
        "description": description,
        "tags": tags.split(","),
        "file_size": len(contents)
    }

# File type validation
@app.post("/upload-image/")
async def upload_image(file: UploadFile = File(...)):
    """
    Validate file type (basic check)
    """
    # Check MIME type
    if not file.content_type.startswith("image/"):
        return {"error": "File must be an image"}
    
    contents = await file.read()
    
    # Check file size (e.g., max 5MB)
    max_size = 5 * 1024 * 1024  # 5MB
    if len(contents) > max_size:
        return {"error": "File too large"}
    
    return {
        "filename": file.filename,
        "size": len(contents),
        "type": file.content_type
    }
```

---

# ADVANCED RESPONSE DETAILS - 2.5 HOURS

## Part 1: Response Models and Serialization

**Formal Definition: Response Model** A Pydantic model that defines the structure and types of the response data. FastAPI uses it to:

1. Validate response data
2. Serialize Python objects to JSON
3. Generate OpenAPI documentation
4. Filter response fields

### Basic Response Model

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class ItemResponse(BaseModel):
    """Model that defines response structure"""
    id: int
    name: str
    price: float
    in_stock: bool

@app.get("/items/{item_id}", response_model=ItemResponse)
def get_item(item_id: int) -> ItemResponse:
    """
    response_model=ItemResponse tells FastAPI:
    1. Validate response against ItemResponse
    2. Serialize to JSON
    3. Generate OpenAPI docs showing ItemResponse
    
    If endpoint returns data that doesn't match,
    FastAPI raises validation error
    """
    return {
        "id": item_id,
        "name": "Widget",
        "price": 9.99,
        "in_stock": True
    }
```

### Response Model Field Exclusion

**Formal Definition: Field Exclusion** Removing certain fields from response even though they exist in the model. Useful for hiding internal details.

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class ItemInDB(BaseModel):
    id: int
    name: str
    price: float
    cost: float  # Internal cost - shouldn't be in response
    margin: float  # Calculated internally

class ItemResponse(BaseModel):
    id: int
    name: str
    price: float
    # cost and margin excluded!

@app.get("/items/{item_id}", response_model=ItemResponse)
def get_item(item_id: int):
    """
    Even if you return cost and margin,
    FastAPI filters them out based on response_model
    """
    return {
        "id": item_id,
        "name": "Widget",
        "price": 9.99,
        "cost": 3.00,      # In database
        "margin": 6.99     # In database
        # But client only sees: id, name, price
    }

# Alternative: Use response_model_exclude
@app.get("/items/{item_id}")
def get_item_v2(item_id: int):
    """
    Exclude specific fields from response
    """
    db_item = {
        "id": item_id,
        "name": "Widget",
        "price": 9.99,
        "cost": 3.00,
        "margin": 6.99
    }
    return db_item

# Flexible exclusion
@app.get("/items/{item_id}")
def get_item_v3(
    item_id: int,
    include_cost: bool = False
):
    """
    Dynamically include/exclude fields based on request
    """
    item = {"id": item_id, "name": "Widget", "price": 9.99, "cost": 3.00}
    
    if not include_cost:
        # Remove cost field
        item.pop("cost", None)
    
    return item
```

### Response Model with Lists

```python
from fastapi import FastAPI
from pydantic import BaseModel
from typing import List

app = FastAPI()

class ItemResponse(BaseModel):
    id: int
    name: str

class OrderResponse(BaseModel):
    id: int
    items: List[ItemResponse]  # List of items
    total: float

@app.get("/orders/{order_id}", response_model=OrderResponse)
def get_order(order_id: int):
    """
    Response model with nested list of models
    Each item validated against ItemResponse
    """
    return {
        "id": order_id,
        "items": [
            {"id": 1, "name": "Widget"},
            {"id": 2, "name": "Gadget"}
        ],
        "total": 29.99
    }
```

---

## Part 2: Custom Response Classes

**Formal Definition: Custom Response** A custom HTTP response object that allows precise control over status code, headers, media type, and body content. Used for non-JSON responses or special cases.

### JSONResponse

```python
from fastapi import FastAPI
from fastapi.responses import JSONResponse

app = FastAPI()

@app.get("/items/")
def read_items():
    """
    Explicitly return JSONResponse for full control
    """
    return JSONResponse(
        status_code=200,
        content={"items": [1, 2, 3]},
        headers={"X-Custom": "header-value"},
        media_type="application/json"
    )

# Custom status code
@app.post("/items/")
def create_item():
    """
    Custom status codes using JSONResponse
    """
    return JSONResponse(
        status_code=201,  # Created
        content={"id": 1, "message": "Item created"},
        headers={"Location": "/items/1"}  # Redirect header
    )
```

### FileResponse

```python
from fastapi import FastAPI
from fastapi.responses import FileResponse
import os

app = FastAPI()

@app.get("/download/{file_name}")
def download_file(file_name: str):
    """
    Return file for download
    Browser prompts user to download file
    """
    file_path = f"/files/{file_name}"
    
    if not os.path.exists(file_path):
        return {"error": "File not found"}
    
    return FileResponse(
        path=file_path,
        media_type="application/octet-stream",  # Binary data
        filename=file_name  # Filename for download
    )

# Specific file types
@app.get("/pdf/{document_name}")
def download_pdf(document_name: str):
    """
    Return PDF file
    """
    return FileResponse(
        path=f"/documents/{document_name}.pdf",
        media_type="application/pdf",
        filename=f"{document_name}.pdf"
    )

@app.get("/image/{image_name}")
def get_image(image_name: str):
    """
    Return image file
    Browser displays image instead of downloading
    """
    return FileResponse(
        path=f"/images/{image_name}.jpg",
        media_type="image/jpeg"
    )
```

### StreamingResponse

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
import io

app = FastAPI()

@app.get("/stream/")
def stream_data():
    """
    Stream large data without loading everything in memory
    Useful for:
    - Large files
    - Real-time data
    - Server-sent events
    """
    
    def generate():
        """Generator function yields data in chunks"""
        for i in range(10):
            yield f"data: chunk {i}\n".encode()
    
    return StreamingResponse(
        generate(),
        media_type="text/event-stream"
    )

# Stream CSV
@app.get("/export-csv/")
def export_csv():
    """
    Stream CSV file without loading in memory
    """
    
    def generate_csv():
        yield b"id,name,price\n"
        for i in range(1000):
            yield f"{i},Item {i},{10.00}\n".encode()
    
    return StreamingResponse(
        generate_csv(),
        media_type="text/csv",
        headers={"Content-Disposition": "attachment; filename=items.csv"}
    )
```

### HTMLResponse

```python
from fastapi import FastAPI
from fastapi.responses import HTMLResponse

app = FastAPI()

@app.get("/", response_class=HTMLResponse)
def get_html():
    """
    Return HTML instead of JSON
    """
    return """
    <html>
        <head>
            <title>My API</title>
        </head>
        <body>
            <h1>Welcome to My API</h1>
            <p>Visit /docs for API documentation</p>
        </body>
    </html>
    """
```

---

## Part 3: Response Status Codes

### Multiple Possible Status Codes

**Formal Definition: Multiple Status Codes** An endpoint that can return different status codes depending on the outcome. Documentation should show all possibilities.

```python
from fastapi import FastAPI, HTTPException, status
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float

# Using status parameter
@app.post("/items/", status_code=status.HTTP_201_CREATED)
def create_item(item: Item):
    """
    201 Created - standard for resource creation
    But might also return:
    - 400 Bad Request (validation error)
    - 409 Conflict (duplicate)
    """
    return item

# Multiple status codes in documentation
from fastapi import Response

@app.get(
    "/items/{item_id}",
    responses={
        200: {"description": "Item found"},
        404: {"description": "Item not found"}
    }
)
def get_item(item_id: int, response: Response):
    """
    OpenAPI documentation shows both possible status codes
    
    responses dict tells FastAPI about alternative responses
    """
    if item_id == 0:
        response.status_code = status.HTTP_404_NOT_FOUND
        return {"error": "Item not found"}
    
    return {"id": item_id, "name": "Widget"}

# Detailed response documentation
@app.post(
    "/items/",
    responses={
        201: {
            "description": "Item created successfully",
            "content": {
                "application/json": {
                    "example": {"id": 1, "name": "Widget", "price": 9.99}
                }
            }
        },
        400: {
            "description": "Invalid input",
            "content": {
                "application/json": {
                    "example": {"detail": "Price must be positive"}
                }
            }
        }
    }
)
def create_item_detailed(item: Item):
    """Document all possible responses with examples"""
    return item
```

---

## Part 4: Response Headers

```python
from fastapi import FastAPI, Response
from fastapi.responses import JSONResponse

app = FastAPI()

# Add headers with Response parameter
@app.get("/items/")
def read_items(response: Response):
    """
    Response parameter allows modifying HTTP response
    """
    response.headers["X-Custom-Header"] = "CustomValue"
    response.headers["Cache-Control"] = "no-cache"
    
    return {"items": [1, 2, 3]}

# Return custom headers with JSONResponse
@app.get("/items/v2/")
def read_items_v2():
    """
    Use JSONResponse for full control over response
    """
    return JSONResponse(
        content={"items": [1, 2, 3]},
        headers={
            "X-Custom": "value",
            "Cache-Control": "max-age=3600"
        }
    )

# Set cookies in headers
@app.post("/login/")
def login(response: Response):
    """
    Set cookies via response headers
    """
    response.set_cookie(
        key="access_token",
        value="fake-token-value",
        max_age=3600,  # Expires in 1 hour
        secure=True,   # HTTPS only
        httponly=True  # Not accessible via JavaScript
    )
    return {"message": "Logged in"}
```

---

# ERROR HANDLING & VALIDATION - 2.5 HOURS

## Part 1: Advanced Error Handling

### Formal Definition: Error Handling

**Error Handling:** The process of detecting, reporting, and responding to errors that occur during API execution. Good error handling:

1. Returns appropriate HTTP status codes
2. Provides clear error messages
3. Includes debugging information (in dev)
4. Handles all error types (validation, database, logic)

### Built-in Validation Errors

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float

@app.post("/items/")
def create_item(item: Item):
    """
    Pydantic automatically validates:
    - Type mismatches (e.g., price="abc" instead of float)
    - Required fields missing
    - Field constraints violated
    
    FastAPI returns 422 Unprocessable Entity with error details:
    {
      "detail": [
        {
          "loc": ["body", "item", "price"],
          "msg": "value is not a valid float",
          "type": "type_error.float"
        }
      ]
    }
    """
    return item
```

### Custom Validation Errors

```python
from fastapi import FastAPI, HTTPException, status
from pydantic import BaseModel, validator

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float
    
    @validator('price')
    def price_must_be_positive(cls, v):
        if v <= 0:
            raise ValueError('Price must be greater than 0')
        return v

@app.post("/items/")
def create_item(item: Item):
    """
    Custom validator raises ValueError
    FastAPI converts to 422 validation error
    """
    return item
```

### HTTPException for Business Logic Errors

```python
from fastapi import FastAPI, HTTPException, status

app = FastAPI()

fake_items_db = {
    1: {"id": 1, "name": "Widget", "owner": "alice"},
    2: {"id": 2, "name": "Gadget", "owner": "bob"}
}

@app.get("/items/{item_id}")
def get_item(item_id: int):
    """
    Use HTTPException for business logic errors
    (not validation errors)
    """
    if item_id not in fake_items_db:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Item {item_id} not found"
        )
    
    return fake_items_db[item_id]

@app.delete("/items/{item_id}")
def delete_item(item_id: int, current_user: str):
    """
    Authorization errors
    """
    if item_id not in fake_items_db:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Item not found"
        )
    
    item = fake_items_db[item_id]
    if item["owner"] != current_user:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="You don't have permission to delete this item"
        )
    
    del fake_items_db[item_id]
    return {"message": "Item deleted"}

@app.post("/items/")
def create_item(name: str):
    """
    Conflict errors
    """
    # Check if item with same name exists
    for item in fake_items_db.values():
        if item["name"].lower() == name.lower():
            raise HTTPException(
                status_code=status.HTTP_409_CONFLICT,
                detail=f"Item '{name}' already exists"
            )
    
    new_id = max(fake_items_db.keys()) + 1
    fake_items_db[new_id] = {"id": new_id, "name": name}
    return fake_items_db[new_id]
```

### Custom Exception Handlers

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse
from fastapi.exceptions import RequestValidationError

app = FastAPI()

# Custom exception class
class ItemNotFoundError(Exception):
    def __init__(self, item_id: int):
        self.item_id = item_id

# Handler for custom exception
@app.exception_handler(ItemNotFoundError)
async def item_not_found_handler(request: Request, exc: ItemNotFoundError):
    """
    When ItemNotFoundError is raised anywhere,
    this handler catches it and returns a response
    """
    return JSONResponse(
        status_code=404,
        content={
            "error": "not_found",
            "detail": f"Item {exc.item_id} not found",
            "item_id": exc.item_id
        }
    )

# Use custom exception
@app.get("/items/{item_id}")
def get_item(item_id: int):
    if item_id == 0:
        raise ItemNotFoundError(item_id)
    return {"id": item_id}

# Override default validation error handler
@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request: Request, exc: RequestValidationError):
    """
    Customize validation error responses
    """
    return JSONResponse(
        status_code=422,
        content={
            "error": "validation_error",
            "detail": str(exc),
            "fields": [
                {
                    "field": ".".join(str(x) for x in error["loc"]),
                    "message": error["msg"]
                }
                for error in exc.errors()
            ]
        }
    )
```

---

## Part 2: Error Response Models

**Formal Definition: Error Response Model** A Pydantic model that defines the structure of error responses. Ensures consistent error format and appears in OpenAPI docs.

```python
from fastapi import FastAPI, HTTPException, status
from pydantic import BaseModel
from typing import Optional

app = FastAPI()

# Define error response structure
class ErrorDetail(BaseModel):
    error_code: str
    message: str
    field: Optional[str] = None

class ErrorResponse(BaseModel):
    status: str = "error"
    errors: list[ErrorDetail]

# Use in responses parameter
@app.get(
    "/items/{item_id}",
    responses={
        200: {"description": "Item found"},
        404: {
            "description": "Item not found",
            "model": ErrorResponse,
            "content": {
                "application/json": {
                    "example": {
                        "status": "error",
                        "errors": [
                            {
                                "error_code": "ITEM_NOT_FOUND",
                                "message": "Item does not exist",
                                "field": "item_id"
                            }
                        ]
                    }
                }
            }
        }
    }
)
def get_item(item_id: int):
    """
    Document error response structure in OpenAPI
    """
    if item_id == 0:
        raise HTTPException(
            status_code=404,
            detail="Item not found"
        )
    return {"id": item_id}
```

---

# DOCUMENTATION & OPENAPI - 1.5 HOURS

## Part 1: OpenAPI Schema Understanding

**Formal Definition: OpenAPI** An open standard for describing REST APIs using JSON/YAML. It specifies:

- Available endpoints
- Request parameters and body
- Response formats
- Authentication methods
- Error responses

**FastAPI automatically generates OpenAPI from your code!**

### How FastAPI Generates OpenAPI

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI(
    title="My API",
    description="A detailed description of what the API does",
    version="1.0.0"
)

class Item(BaseModel):
    """Item model - description shows in OpenAPI"""
    id: int = Field(..., description="The item ID")
    name: str = Field(..., min_length=1, description="The item name")
    price: float = Field(..., gt=0, description="Price in USD")

@app.get("/items/{item_id}")
def get_item(item_id: int = Path(..., description="Item ID to retrieve")):
    """
    Get an item by ID.
    
    This docstring appears in OpenAPI as:
    - Summary: First line
    - Description: Full docstring
    
    All type hints, Field descriptions, constraints
    automatically added to OpenAPI schema
    """
    return {"id": item_id}

# Access generated OpenAPI at:
# GET /openapi.json returns full OpenAPI specification
```

### OpenAPI Auto-Documentation Features

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

class Item(BaseModel):
    name: str = Field(
        ...,
        min_length=1,
        max_length=100,
        description="Item name",
        example="Widget"
    )
    price: float = Field(
        ...,
        gt=0,
        description="Price in USD",
        example=9.99
    )

@app.post(
    "/items/",
    summary="Create Item",
    responses={
        201: {"description": "Item created successfully"},
        400: {"description": "Invalid input"}
    }
)
def create_item(item: Item):
    """
    Create a new item.
    
    This detailed description explains:
    - What the endpoint does
    - Business logic
    - Special cases
    
    Features in OpenAPI:
    - Type hints → schema types
    - Field descriptions → field documentation
    - Examples → sample values
    - responses → possible response codes
    """
    return item
```

---

## Part 2: Interactive Documentation

### Swagger UI (/docs)

```python
from fastapi import FastAPI

app = FastAPI()

# Automatically available at /docs
# Features:
# - Interactive endpoint explorer
# - Try-it-out functionality
# - Request/response examples
# - All parameter types shown
# - Dropdown for enum values

@app.get("/items/")
def read_items():
    """
    In /docs, users can:
    1. See this description
    2. See all parameters
    3. Click "Try it out"
    4. Edit request
    5. Click "Execute"
    6. See response
    """
    return {"items": [1, 2, 3]}
```

### ReDoc (/redoc)

```python
# Automatically available at /redoc
# Alternative documentation format:
# - Better for reading
# - Search functionality
# - Organized by tags
# - Reference-style documentation

@app.get("/items/", tags=["items"])
def read_items():
    """Lists all items"""
    return {"items": [1, 2, 3]}

@app.post("/items/", tags=["items"])
def create_item():
    """Creates new item"""
    return {"id": 1}
```

### Custom Documentation Configuration

```python
from fastapi import FastAPI
from fastapi.openapi.utils import get_openapi

app = FastAPI(
    title="My API",
    description="Complete API description",
    version="1.0.0",
    docs_url="/documentation",  # Custom /docs URL
    redoc_url="/reference",     # Custom /redoc URL
    openapi_url="/api.json"     # Custom OpenAPI spec URL
)

# Disable documentation
app_no_docs = FastAPI(
    docs_url=None,    # Disable Swagger
    redoc_url=None    # Disable ReDoc
)

# Custom OpenAPI schema
def custom_openapi():
    if app.openapi_schema:
        return app.openapi_schema
    
    openapi_schema = get_openapi(
        title="My Custom API",
        version="2.0.0",
        routes=app.routes,
    )
    
    # Modify schema
    openapi_schema["info"]["x-logo"] = {
        "url": "https://example.com/logo.png"
    }
    
    app.openapi_schema = openapi_schema
    return app.openapi_schema

app.openapi = custom_openapi
```

---

# WEEK 3 CHECKPOINT PROJECT

## Project: Blog API with Advanced Features

### Objective

Build a comprehensive Blog API demonstrating:

1. Advanced path parameters with validation
2. Complex request handling (multiple body params, files)
3. Advanced response handling (multiple status codes, custom responses)
4. Comprehensive error handling
5. Complete OpenAPI documentation

### Project Requirements

#### 1. Data Models

```python
# Models to create:
class Tag(BaseModel):
    id: int
    name: str = Field(..., min_length=1, max_length=50)

class Author(BaseModel):
    id: int
    name: str = Field(..., min_length=3, max_length=100)
    email: str = Field(..., regex=r"^[\w\.-]+@[\w\.-]+\.\w+$")
    bio: Optional[str] = Field(None, max_length=500)

class PostCreate(BaseModel):
    title: str = Field(..., min_length=5, max_length=200)
    content: str = Field(..., min_length=10, max_length=10000)
    author_id: int = Field(..., gt=0)
    tags: List[int] = []
    published: bool = False

class PostUpdate(BaseModel):
    title: Optional[str] = Field(None, min_length=5, max_length=200)
    content: Optional[str] = Field(None, min_length=10)
    published: Optional[bool] = None

class PostResponse(BaseModel):
    id: int
    title: str
    content: str
    author: Author
    tags: List[Tag]
    published: bool
    views: int
    created_at: str
```

#### 2. API Endpoints to Implement

**Posts Endpoints:**

1. **GET /posts/** - List posts with pagination
    
    - Query params: skip, limit
    - Query param: published (filter by publication status)
    - Response: List[PostResponse]
2. **GET /posts/{post_id}** - Get specific post
    
    - Path param: post_id (int, > 0)
    - Response: PostResponse
    - Error: 404 if not found
3. **GET /posts/by-slug/{slug}** - Get by slug
    
    - Path param: slug (str, regex validation)
    - Response: PostResponse
4. **POST /posts/** - Create post
    
    - Body: PostCreate model
    - Response: PostResponse
    - Status: 201 Created
5. **PUT /posts/{post_id}** - Replace post
    
    - Path param: post_id
    - Body: PostCreate
    - Response: PostResponse
6. **PATCH /posts/{post_id}** - Update post
    
    - Path param: post_id
    - Body: PostUpdate
    - Response: PostResponse
7. **DELETE /posts/{post_id}** - Delete post
    
    - Path param: post_id
    - Status: 204 No Content

**Advanced Endpoints:**

8. **POST /posts/{post_id}/upload-image** - Upload featured image
    
    - Path param: post_id
    - File param: image (UploadFile)
    - Response: {"message": "Image uploaded"}
9. **GET /posts/{post_id}/comments** - Get post comments
    
    - With pagination
    - Response: List with CommentResponse
10. **POST /posts/search/** - Search posts
    
    - Query param: q (search query)
    - Query param: tags (comma-separated)
    - Response: List[PostResponse]

#### 3. Validation Requirements

- Title: 5-200 characters
- Content: 10-10000 characters
- Email: Valid email format
- Author ID: Must be positive
- Post ID: Must be positive
- Slug: Regex validation (alphanumeric, hyphens only)

#### 4. Error Handling

Implement custom error handling for:

- 404: Post not found
- 404: Author not found
- 400: Invalid request data
- 409: Duplicate post slug
- 413: File too large
- 415: Unsupported media type

#### 5. Documentation Requirements

- Docstrings for all endpoints
- Descriptions for all parameters
- Example values for models
- Documented response status codes
- Documented error responses

#### 6. Interactive Documentation

- All endpoints visible in /docs
- Descriptions show constraints
- Examples shown in forms
- Try-it-out works for all endpoints

### Deliverables

1. **main.py** - Complete FastAPI application
2. **models.py** - All Pydantic models (separate file)
3. **README.md** - Setup and API documentation
4. **requirements.txt** - All dependencies

### Testing Checklist

- [ ] List posts with pagination
- [ ] Get specific post
- [ ] Create post (validate constraints)
- [ ] Update post (PUT and PATCH)
- [ ] Delete post
- [ ] Upload image file
- [ ] Search posts
- [ ] All error cases return correct status codes
- [ ] All endpoints documented in /docs
- [ ] Field validation works correctly

---

# EXERCISE SOLUTIONS

## Path Parameter Validation Exercises

### Exercise 1: Numeric Constraints

**Exercise:**

```python
# Create endpoint: GET /products/{product_id}
# - product_id must be 1-10000
# Create endpoint: GET /ratings/{rating}
# - rating must be 1-5 (inclusive)
```

**Solution:**

```python
from fastapi import Path

@app.get("/products/{product_id}")
def get_product(product_id: int = Path(..., ge=1, le=10000)):
    """Get product with ID validation (1-10000)"""
    return {"product_id": product_id}

@app.get("/ratings/{rating}")
def get_rating(rating: int = Path(..., ge=1, le=5)):
    """Get rating (1-5 stars)"""
    return {"rating": rating}
```

### Exercise 2: String Validation with Regex

**Exercise:**

```python
# Create endpoint: GET /codes/{code}
# - code: format ABC1234 (3 letters, 4 numbers)
```

**Solution:**

```python
from fastapi import Path

@app.get("/codes/{code}")
def get_code(code: str = Path(..., regex="^[A-Z]{3}[0-9]{4}$")):
    """Get code with validation"""
    return {"code": code}
```

## Request Body Exercises

### Exercise 1: Multiple Body Parameters

**Exercise:**

```python
# Create endpoint: POST /orders/
# - item (Pydantic model)
# - quantity (int, > 0)
# - discount (float, 0-100, optional)
```

**Solution:**

```python
from fastapi import Body
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    price: float

@app.post("/orders/")
def create_order(
    item: Item = Body(...),
    quantity: int = Body(..., gt=0),
    discount: float = Body(0, ge=0, le=100)
):
    """Create order with item, quantity, discount"""
    total = (item.price * quantity) * (1 - discount/100)
    return {"item": item, "quantity": quantity, "total": total}
```

## Error Handling Exercises

### Exercise 1: Custom Exceptions

**Exercise:**

```python
# Create custom exception: OutOfStockError
# Create handler that returns 409 Conflict
```

**Solution:**

```python
class OutOfStockError(Exception):
    def __init__(self, item_id: int):
        self.item_id = item_id

@app.exception_handler(OutOfStockError)
async def out_of_stock_handler(request, exc):
    return JSONResponse(
        status_code=409,
        content={"error": "out_of_stock", "item_id": exc.item_id}
    )

@app.post("/purchase/")
def purchase(item_id: int):
    stock = {1: 5, 2: 0, 3: 10}
    if stock[item_id] == 0:
        raise OutOfStockError(item_id)
    return {"success": True}
```

---

# SUMMARY AND CHECKLIST

## Week 3 Learning Path Summary

**Total Time: 12-13 Hours**

### Path Operations Deep Dive (2.5 hours)

- ✅ Path parameter type conversion
- ✅ Numeric constraints (gt, gte, lt, lte)
- ✅ String constraints (min_length, max_length, regex)
- ✅ Enum path parameters
- ✅ Multiple path parameters with validation
- ✅ Path parameters in documentation

### Advanced Request Details (3 hours)

- ✅ Multiple body parameters
- ✅ Body() for explicit body declaration
- ✅ Form data handling
- ✅ File uploads (single and multiple)
- ✅ File validation
- ✅ Mixed request types

### Advanced Response Details (2.5 hours)

- ✅ Response models and serialization
- ✅ Field exclusion from responses
- ✅ Custom response classes (JSON, File, HTML)
- ✅ Multiple status codes
- ✅ Response headers and cookies
- ✅ Streaming responses

### Error Handling & Validation (2.5 hours)

- ✅ Built-in validation errors
- ✅ Custom validation in models
- ✅ HTTPException for business logic errors
- ✅ Custom exception handlers
- ✅ Error response models
- ✅ Standardized error format

### Documentation & OpenAPI (1.5 hours)

- ✅ OpenAPI schema generation
- ✅ Swagger UI (/docs)
- ✅ ReDoc (/redoc)
- ✅ Field descriptions and examples
- ✅ Response documentation
- ✅ Custom OpenAPI configuration

## Week 3 Success Criteria

By end of Week 3, you should be able to:

1. **Validate any path parameter** - Using constraints, regex, enums
2. **Handle complex requests** - Multiple body params, files, forms
3. **Control responses precisely** - Status codes, headers, custom classes
4. **Handle errors comprehensively** - Validation, business logic, custom exceptions
5. **Write self-documenting APIs** - Full OpenAPI documentation automatically
6. **Test in interactive UI** - /docs endpoint shows everything

## Key Takeaways

- **Validation is automatic** - Use type hints and Field constraints
- **Constraints appear in docs** - Constraints auto-document themselves
- **Errors should be informative** - Help users understand what went wrong
- **Responses should be typed** - response_model ensures consistency
- **Documentation is free** - OpenAPI generated automatically
- **Multiple formats available** - JSON, files, HTML, streams

## Common Mistakes to Avoid

1. ❌ Not using Path() for path parameter constraints
2. ❌ Mixing Body() and query parameters incorrectly
3. ❌ Not handling 404 errors
4. ❌ Returning inconsistent error formats
5. ❌ Forgetting response_model
6. ❌ Not validating file uploads
7. ❌ Ignoring status codes (should use 201 for POST, 204 for DELETE)

## Next Steps

After Week 3, you have comprehensive understanding of request/response handling. Week 4 will introduce dependencies in depth.

**Before moving to Week 4:**

- [ ] Build Blog API checkpoint project
- [ ] All endpoints work correctly
- [ ] All validation rules enforced
- [ ] Error handling comprehensive
- [ ] OpenAPI documentation complete
- [ ] All endpoints testable in /docs

---

# ADDITIONAL RESOURCES

## Official Documentation

- FastAPI Path Parameters: https://fastapi.tiangolo.com/tutorial/path-params/
- FastAPI Request Body: https://fastapi.tiangolo.com/tutorial/body/
- FastAPI Responses: https://fastapi.tiangolo.com/advanced/response-files/
- FastAPI Exception Handling: https://fastapi.tiangolo.com/tutorial/handling-errors/

## Tools

- Postman: Test complex requests with files
- Thunder Client: VS Code extension for API testing
- curl: Command-line testing

## Testing File Uploads

```bash
# Test file upload with curl
curl -X POST "http://localhost:8000/upload/" \
  -F "file=@myfile.txt"

# Test with multiple files
curl -X POST "http://localhost:8000/upload-multiple/" \
  -F "files=@file1.txt" \
  -F "files=@file2.txt"

# Test with form fields
curl -X POST "http://localhost:8000/upload-with-metadata/" \
  -F "file=@image.jpg" \
  -F "description=My image" \
  -F "tags=photo,nature"
```

---

**End of Week 3 Content**

Excellent progress! You now have deep expertise in request/response handling and validation. Next week introduces sophisticated dependency patterns. 🚀