# 1. Function Decorators Basics

A decorator is just:

- A function
- That takes another function as input
- Returns a modified version of that function

```python
def logger(func):
    def wrapper():
        print("Function started")
        func()
        print("Function ended")
    return wrapper


@logger
def greet():
    print("Hello")


greet()
```

Output:

```
Function started
Hello
Function ended
```

The line:

```
@logger
def greet():
```

is equivalent to:

```
def greet():
    print("Hello")

greet = logger(greet)
```

---

## Handling arguments

Your first decorator probably breaks here:

```
@logger
def add(a, b):
    return a + b
```

Because `wrapper()` takes no arguments.

Solution:

```
def logger(func):
    def wrapper(*args, **kwargs):
        print("Calling function")
        result = func(*args, **kwargs)
        print("Finished")
        return result
    return wrapper
```

Now it works for any function.

```
@logger
def add(a, b):
    return a + b

print(add(2, 3))
```

---

## Preserving metadata

Without this:

```
print(add.__name__)
```

you get:

```
wrapper
```

instead of:

```
add
```

Use `functools.wraps`.

```
from functools import wraps

def logger(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("Calling")
        return func(*args, **kwargs)
    return wrapper
```

This is standard practice.

---

# 2. Decorators With Arguments

Sometimes the decorator itself needs configuration.

Example:

```
@repeat(3)
def hello():
    print("Hello")
```

This means:

```
hello = repeat(3)(hello)
```

Notice there are now **three layers**.

```
def repeat(times):

    def decorator(func):

        def wrapper(*args, **kwargs):
            for _ in range(times):
                func(*args, **kwargs)

        return wrapper

    return decorator
```

Usage:

```
@repeat(3)
def hello():
    print("Hi")

hello()
```

Output:

```
Hi
Hi
Hi
```

---

## Structure to memorize

```
def decorator_args(arg1):

    def actual_decorator(func):

        def wrapper(*args, **kwargs):
            ...
            return func(*args, **kwargs)

        return wrapper

    return actual_decorator
```

Think:

```
arguments -> decorator -> wrapper
```

---

## Example: Permission system

```
def require_role(role):
    def decorator(func):
        def wrapper(user):
            if user["role"] != role:
                print("Access denied")
                return

            return func(user)

        return wrapper
    return decorator
```

```
@require_role("admin")
def delete_database(user):
    print("Database deleted")
```

---

# 3. Stacking Decorators

Multiple decorators execute from bottom to top.

```
@decorator1
@decorator2
def func():
    pass
```

means:

```
func = decorator1(decorator2(func))
```

---

Example:

```
def bold(func):
    def wrapper():
        return f"<b>{func()}</b>"
    return wrapper


def italic(func):
    def wrapper():
        return f"<i>{func()}</i>"
    return wrapper


@bold
@italic
def text():
    return "Hello"

print(text())
```

Execution:

```
text
↓
italic(text)
↓
bold(result)
```

Output:

```
<b><i>Hello</i></b>
```

---

Another example:

```
@cache
@logger
def expensive_function():
    ...
```

Execution order:

```
cache
    └── logger
            └── function
```

So cache runs first.

---

# 4. Practical Decorator Patterns Used in FastAPI

This is where you'll see decorators constantly.

---

## Route registration

```
@app.get("/users")
def get_users():
    return ["Alice", "Bob"]
```

`@app.get()` is a decorator.

Internally FastAPI does roughly this:

```
def get(path):
    def decorator(func):
        routes[path] = func
        return func
    return decorator
```

So:

```
@app.get("/users")
def get_users():
    ...
```

becomes:

```
get_users = app.get("/users")(get_users)
```

FastAPI stores the function and calls it when a request arrives.

---

## Dependency Injection

```
@app.get("/profile")
def profile(user=Depends(get_current_user)):
    return user
```

Not technically a decorator, but built using similar ideas of wrapping and injecting behavior around functions.

---

## Authentication Decorator Example

```
def login_required(func):

    @wraps(func)
    def wrapper(*args, **kwargs):
        token = kwargs.get("token")

        if token != "secret":
            raise Exception("Unauthorized")

        return func(*args, **kwargs)

    return wrapper
```

```
@login_required
def get_secret_data(token):
    return "Top secret"
```

---

## Timing decorator

Very common for APIs.

```
import time
from functools import wraps

def timing(func):

    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()

        result = func(*args, **kwargs)

        end = time.time()

        print(f"Took {end - start:.4f}s")

        return result

    return wrapper
```

---

## Caching decorator

```
from functools import wraps

def cache(func):
    memory = {}

    @wraps(func)
    def wrapper(x):
        if x in memory:
            return memory[x]

        result = func(x)
        memory[x] = result
        return result

    return wrapper
```

---

## Rate limiting decorator

```
def limit_requests(max_requests):
    count = 0

    def decorator(func):
        def wrapper(*args, **kwargs):
            nonlocal count

            if count >= max_requests:
                raise Exception("Rate limit exceeded")

            count += 1
            return func(*args, **kwargs)

        return wrapper

    return decorator
```

---

# Mental Model

Almost every decorator follows one of these templates:

### Simple decorator

```
def decorator(func):
    def wrapper(*args, **kwargs):
        # before
        result = func(*args, **kwargs)
        # after
        return result
    return wrapper
```

### Decorator with arguments

```
def decorator_args(config):
    def decorator(func):
        def wrapper(*args, **kwargs):
            ...
            return func(*args, **kwargs)
        return wrapper
    return decorator
```

### Stacked decorators

```
outer(
    middle(
        inner(
            function
        )
    )
)
```

If you understand those three patterns, you've covered about 95% of decorator usage in real Python codebases including FastAPI, Flask, Django, and many libraries.