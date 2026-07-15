# FastAPI

Python framework for developing Web APIs.

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def home():
    return "Welcome"
```

## Parameters

### Query Parameters

Query parameters come after `?` in the URL.

Use query parameters for:

* Filtering
* Searching
* Sorting
* Pagination
* Optional settings

Multiple query parameters are:

* Written after `?`
* Separated by `&`

Example:

```text
http://localhost:8000/products?id=1&title=mobile
```

Here, `id` and `title` are **query parameters**.

---

### Path Parameters

Path parameters are part of the URL path itself.

Use path parameters to identify which resource the client wants.

Example route:

```text
http://localhost:8000/products/{product_id}
```

Example request:

```text
http://localhost:8000/products/15
```

Here, `product_id` is a **path parameter**.

```python
@app.get("/products/{product_id}")
def get_one_product(product_id: int):
    for prod in products:
        if prod["id"] == product_id:
            return prod

    return []
```

Example request:

```text
http://localhost:8000/products/100
```

Here, `100` is the **path parameter value**.

If a product with the given `product_id` exists, that product is returned; otherwise, an empty array is returned.

---

## HTTP Methods

* `GET` → Retrieve data from the server.
* `POST` → Create a new resource on the server.
* `PUT` → Modify an existing resource on the server.
* `DELETE` → Delete a resource from the server.

These operations are known as **CRUD**:

* **Create** → POST
* **Read** → GET
* **Update** → PUT
* **Delete** → DELETE

---

## Accessing Query Parameters

```python
from fastapi import Request

@app.get("/greet")
def greet(request: Request):
    query_params = dict(request.query_params)

    print(query_params)

    return (
        f"You are {query_params.get('name')}, "
        f"you are {query_params.get('age')} years old"
    )
```
