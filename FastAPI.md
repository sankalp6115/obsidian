```python
from fastapi import FastAPI
app = FastAPI()

@app.get("/")
def home():
	return "Welcome"
```
Parameters
https://localhost:8000/products?id=1&title=mobile
id, title are **Query Parameters**

https:/localhost:8000/products/100
100 is **Path Parameter**


```python
# Path param
@app.get("/products/{product_id}")
def get_one_product(product_id:int):
	product = None
	for prod in products:
		if prod["id"] == product_id:
		return prod
	return []
```

If product with path parameter id is available, the product is shown, else an empty array is returned.


get - to get something from server
post- to create new resource in server
put - to modify resource in server
delete - to delete resource from server
CRUD
