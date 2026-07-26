**Duration:** 12-13 hours  
**Learning Style:** High detail with extensive code examples and exercises  
**Goal:** Master decorators and async/await before FastAPI

---

## Table of Contents

1. [Decorators - 3 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#decorators---3-hours)
2. [Async/Await & Concurrency - 4 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#asyncawait--concurrency---4-hours)
3. [Type Hints Deep Dive - 3 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#type-hints-deep-dive---3-hours)
4. [Built-in Tools - 2 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#built-in-tools---2-hours)
5. [Week 1 Checkpoint Project](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#week-1-checkpoint-project)
6. [Exercise Solutions](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#exercise-solutions)

---

# DECORATORS - 3 HOURS

## What Are Decorators? (Understanding the Concept)

A **decorator** is a function that **takes another function as input, modifies or enhances its behavior, and returns a new function**. Decorators are a form of "function wrapping."

### Why Do We Need Decorators?

Imagine you have 10 functions and you want to add logging to all of them. Without decorators:

```python
# ❌ BAD: Repetitive code
def function1():
    print("Logging function1")
    print("Function1 executing")
    print("Logging function1 done")

def function2():
    print("Logging function2")
    print("Function2 executing")
    print("Logging function2 done")

def function3():
    print("Logging function3")
    print("Function3 executing")
    print("Logging function3 done")

# This violates DRY principle (Don't Repeat Yourself)
```

With decorators:

```python
# ✅ GOOD: DRY approach
def logging_decorator(func):
    def wrapper():
        print(f"Logging {func.__name__}")
        result = func()
        print(f"Logging {func.__name__} done")
        return result
    return wrapper

@logging_decorator
def function1():
    print("Function1 executing")

@logging_decorator
def function2():
    print("Function2 executing")

@logging_decorator
def function3():
    print("Function3 executing")

# All functions now have logging without code duplication!
```

---

## Part 1: Basic Function Decorators (1.5 hours)

### 1.1: Simple Decorator Without Arguments

The simplest form - a decorator that takes a function and wraps it:

```python
def my_decorator(func):
    """
    A simple decorator that prints before and after function execution
    """
    def wrapper():
        print("Something before the function is called")
        result = func()
        print("Something after the function is called")
        return result
    return wrapper

@my_decorator
def say_hello():
    print("Hello!")

# Calling the decorated function
say_hello()

# OUTPUT:
# Something before the function is called
# Hello!
# Something after the function is called
```

**How it works step by step:**

```python
# Step 1: Python sees the @ decorator syntax
@my_decorator
def say_hello():
    print("Hello!")

# Step 2: Python transforms it to:
def say_hello():
    print("Hello!")
say_hello = my_decorator(say_hello)  # say_hello is now the wrapper function!

# Step 3: When you call say_hello(), you're actually calling the wrapper
```

### 1.2: Decorator With Arguments to the Function

Functions often have arguments. We need decorators to handle them:

```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        # *args captures positional arguments
        # **kwargs captures keyword arguments
        print(f"Calling function: {func.__name__}")
        print(f"With args: {args} and kwargs: {kwargs}")
        result = func(*args, **kwargs)
        return result
    return wrapper

@my_decorator
def add(a, b):
    """Add two numbers"""
    return a + b

@my_decorator
def greet(name, greeting="Hello"):
    """Greet someone"""
    return f"{greeting}, {name}!"

# Test
result1 = add(5, 3)
print(f"Result: {result1}\n")

result2 = greet("Alice", greeting="Hi")
print(f"Result: {result2}")

# OUTPUT:
# Calling function: add
# With args: (5, 3) and kwargs: {}
# Result: 8
#
# Calling function: greet
# With args: ('Alice',) and kwargs: {'greeting': 'Hi'}
# Result: Hi, Alice!
```

**Understanding \*args and **kwargs:**

```python
def example_func(*args, **kwargs):
    print(f"Positional args: {args}")
    print(f"Keyword args: {kwargs}")

example_func(1, 2, 3, name="Alice", age=30)
# OUTPUT:
# Positional args: (1, 2, 3)
# Keyword args: {'name': 'Alice', 'age': 30}
```

### 1.3: Real-World Example - Timing Decorator

```python
import time

def timing_decorator(func):
    """Decorator that measures how long a function takes to execute"""
    def wrapper(*args, **kwargs):
        start_time = time.time()
        result = func(*args, **kwargs)
        end_time = time.time()
        elapsed = end_time - start_time
        print(f"{func.__name__} took {elapsed:.4f} seconds")
        return result
    return wrapper

@timing_decorator
def slow_function():
    """Simulates a slow operation"""
    time.sleep(2)
    print("Function completed!")

slow_function()

# OUTPUT:
# Function completed!
# slow_function took 2.0032 seconds
```

---

## Part 2: Decorators With Arguments (1 hour)

Sometimes you want to customize the decorator's behavior. For that, you need a decorator that takes arguments.

### 2.1: Basic Decorator With Arguments

```python
def repeat_decorator(times):
    """
    Decorator that repeats function execution N times
    This is a decorator FACTORY (returns a decorator)
    """
    def actual_decorator(func):
        def wrapper(*args, **kwargs):
            results = []
            for i in range(times):
                print(f"Execution {i+1}")
                result = func(*args, **kwargs)
                results.append(result)
            return results
        return wrapper
    return actual_decorator

@repeat_decorator(times=3)
def greet(name):
    return f"Hello, {name}!"

result = greet("Alice")
print(result)

# OUTPUT:
# Execution 1
# Execution 2
# Execution 3
# ['Hello, Alice!', 'Hello, Alice!', 'Hello, Alice!']
```

**Understanding the flow:**

```python
# When Python sees this:
@repeat_decorator(times=3)
def greet(name):
    return f"Hello, {name}!"

# It does this:
def greet(name):
    return f"Hello, {name}!"
greet = repeat_decorator(times=3)(greet)

# Breaking it down:
# Step 1: repeat_decorator(times=3) returns the actual_decorator function
# Step 2: actual_decorator(greet) returns the wrapper function
# Step 3: greet is now the wrapper function
```

### 2.2: Practical Example - Rate Limiting Decorator

```python
import time

def rate_limit(max_calls, time_window):
    """
    Decorator that limits how many times a function can be called
    within a specific time window
    """
    def actual_decorator(func):
        calls = []
        
        def wrapper(*args, **kwargs):
            now = time.time()
            
            # Remove old calls outside the time window
            calls[:] = [call_time for call_time in calls 
                       if call_time > now - time_window]
            
            # Check if we've exceeded the limit
            if len(calls) >= max_calls:
                raise Exception(f"Rate limit exceeded: {max_calls} calls per {time_window}s")
            
            # Record this call
            calls.append(now)
            
            # Execute the function
            return func(*args, **kwargs)
        
        return wrapper
    return actual_decorator

@rate_limit(max_calls=3, time_window=5)
def api_call():
    return "API call successful!"

# Try calling it
for i in range(4):
    try:
        print(f"Call {i+1}: {api_call()}")
    except Exception as e:
        print(f"Call {i+1}: {e}")

# OUTPUT:
# Call 1: API call successful!
# Call 2: API call successful!
# Call 3: API call successful!
# Call 4: Rate limit exceeded: 3 calls per 5s
```

---

## Part 3: Stacking Decorators (0.5 hours)

You can apply multiple decorators to the same function. They execute from bottom to top:

```python
def decorator_a(func):
    def wrapper(*args, **kwargs):
        print("A Start")
        result = func(*args, **kwargs)
        print("A End")
        return result
    return wrapper

def decorator_b(func):
    def wrapper(*args, **kwargs):
        print("B Start")
        result = func(*args, **kwargs)
        print("B End")
        return result
    return wrapper

@decorator_a
@decorator_b
def hello():
    print("Hello!")

hello()

# OUTPUT:
# A Start
# B Start
# Hello!
# B End
# A End

# The order is: A wraps B, which wraps hello()
# So when calling: A's wrapper -> B's wrapper -> hello()
```

**More practical example - combining logging and timing:**

```python
import time

def timing(func):
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"Took {time.time() - start:.4f}s")
        return result
    return wrapper

def logging(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        result = func(*args, **kwargs)
        print(f"Finished {func.__name__}")
        return result
    return wrapper

@timing
@logging
def process_data(n):
    time.sleep(0.5)
    return f"Processed {n} items"

result = process_data(100)
print(result)

# OUTPUT:
# Calling process_data
# Finished process_data
# Took 0.5002s
# Processed 100 items
```

---

## Part 4: Preserving Function Metadata (Important!)

Here's a problem: when you decorate a function, it loses its original name and docstring:

```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@my_decorator
def greet(name):
    """Greet someone by name"""
    return f"Hello, {name}!"

print(greet.__name__)       # OUTPUT: wrapper (WRONG! Should be 'greet')
print(greet.__doc__)        # OUTPUT: None (WRONG! Should be the docstring)
```

**Solution: Use `functools.wraps`**

```python
from functools import wraps

def my_decorator(func):
    @wraps(func)  # This preserves func's metadata
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@my_decorator
def greet(name):
    """Greet someone by name"""
    return f"Hello, {name}!"

print(greet.__name__)       # OUTPUT: greet ✓
print(greet.__doc__)        # OUTPUT: Greet someone by name ✓
```

**This is CRITICAL for FastAPI!** FastAPI uses function metadata to generate documentation, so always use `@wraps`.

---

## Part 5: Practical Decorators for FastAPI (0.5 hours)

### Example 1: Validation Decorator

```python
from functools import wraps

def validate_positive(param_index):
    """Validates that a numeric parameter is positive"""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            if args[param_index] <= 0:
                raise ValueError(f"Parameter at index {param_index} must be positive")
            return func(*args, **kwargs)
        return wrapper
    return decorator

@validate_positive(0)  # Validate first parameter
def calculate_square_root(number):
    return number ** 0.5

# Test
print(calculate_square_root(16))  # OUTPUT: 4.0

try:
    calculate_square_root(-4)  # Raises ValueError
except ValueError as e:
    print(f"Error: {e}")
```

### Example 2: Authentication Decorator (FastAPI pattern)

```python
from functools import wraps
from typing import Optional

def require_auth(allowed_roles: list = None):
    """Decorator that checks if user has required role"""
    if allowed_roles is None:
        allowed_roles = ["user"]
    
    def decorator(func):
        @wraps(func)
        def wrapper(current_user, *args, **kwargs):
            if current_user is None:
                raise PermissionError("Authentication required")
            if current_user.role not in allowed_roles:
                raise PermissionError(f"Required role: {allowed_roles}")
            return func(current_user, *args, **kwargs)
        return wrapper
    return decorator

class User:
    def __init__(self, name, role):
        self.name = name
        self.role = role

@require_auth(allowed_roles=["admin"])
def delete_user(current_user, user_id):
    return f"{current_user.name} deleted user {user_id}"

# Test
admin = User("Admin Alice", "admin")
regular = User("Regular Bob", "user")

print(delete_user(admin, 123))  # Works

try:
    delete_user(regular, 123)  # Raises PermissionError
except PermissionError as e:
    print(f"Error: {e}")
```

---

## Decorators Summary Table

|Type|Use Case|Complexity|
|---|---|---|
|Simple decorator|Add logging, timing|Easy|
|Decorator with args|Customizable behavior|Medium|
|Stacked decorators|Multiple enhancements|Medium|
|Parameterized decorator|Role-based access|Hard|

---

## Exercises: Decorators

### Exercise 1: Simple Logger Decorator

Create a decorator that logs the function name, arguments, and return value.

```python
# Your solution here
def logger(func):
    pass

@logger
def add(a, b):
    return a + b

add(5, 3)
# Expected output:
# Calling add with args (5, 3)
# add returned 8
```

### Exercise 2: Cache/Memoization Decorator

Create a decorator that caches function results to avoid recalculation.

```python
# Your solution here
def cache_decorator(func):
    pass

@cache_decorator
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

print(fibonacci(10))  # Should be instant even on second call
```

### Exercise 3: Parameterized Validator Decorator

Create a decorator that validates function arguments meet certain conditions.

```python
# Your solution here
def validate_types(**types):
    pass

@validate_types(a=int, b=int)
def multiply(a, b):
    return a * b

multiply(3, 4)  # Works
multiply("3", "4")  # Should raise TypeError
```

**See Exercise Solutions at end of document**

---

# ASYNC/AWAIT & CONCURRENCY - 4 HOURS

## Understanding Synchronous vs Asynchronous Code

### Synchronous Code (Blocking)

```python
import time

def task1():
    print("Task 1 starting")
    time.sleep(2)
    print("Task 1 done")

def task2():
    print("Task 2 starting")
    time.sleep(2)
    print("Task 2 done")

# Running synchronously
start = time.time()
task1()
task2()
elapsed = time.time() - start
print(f"Total time: {elapsed:.1f}s")

# OUTPUT:
# Task 1 starting
# Task 1 done
# Task 2 starting
# Task 2 done
# Total time: 4.0s (SLOW! Tasks run one after another)
```

The problem: **Task 2 starts only after Task 1 finishes.** If we're waiting for I/O (network, database), the CPU is idle—we're wasting time.

### Asynchronous Code (Non-blocking)

```python
import asyncio

async def task1():
    print("Task 1 starting")
    await asyncio.sleep(2)  # Yields control while waiting
    print("Task 1 done")

async def task2():
    print("Task 2 starting")
    await asyncio.sleep(2)  # Yields control while waiting
    print("Task 2 done")

# Running asynchronously
async def main():
    start = time.time()
    await asyncio.gather(task1(), task2())  # Run concurrently
    elapsed = time.time() - start
    print(f"Total time: {elapsed:.1f}s")

asyncio.run(main())

# OUTPUT:
# Task 1 starting
# Task 2 starting
# Task 1 done
# Task 2 done
# Total time: 2.0s (FAST! Tasks run concurrently)
```

**Key insight:** While Task 1 is waiting, Task 2 can run. We go from 4 seconds to 2 seconds!

---

## Part 1: Async/Await Fundamentals (1.5 hours)

### 1.1: Understanding async def

```python
import asyncio

# Regular function
def regular_function():
    return "I'm regular"

# Async function
async def async_function():
    return "I'm async"

# Key difference:
print(regular_function())  # Executes immediately, returns "I'm regular"
print(async_function())    # Returns a coroutine object, NOT the result!

# OUTPUT:
# I'm regular
# <coroutine object async_function at 0x...>

# To actually run an async function, you need an event loop:
result = asyncio.run(async_function())
print(result)  # Now we get "I'm async"
```

**What is a coroutine?** It's an object that represents the async function. It doesn't execute until you `await` it or use `asyncio.run()`.

### 1.2: Understanding await

```python
import asyncio

async def fetch_data():
    print("Fetching data...")
    await asyncio.sleep(2)  # Simulates network call
    print("Data fetched!")
    return {"user": "Alice", "age": 30}

async def main():
    # Method 1: await - waits for the result
    result = await fetch_data()
    print(f"Result: {result}")

asyncio.run(main())

# OUTPUT:
# Fetching data...
# Data fetched!
# Result: {'user': 'Alice', 'age': 30}
```

**Key point:** You can only use `await` inside an `async def` function. Trying to use `await` outside an async context will raise a `SyntaxError`.

```python
# ❌ WRONG - This raises SyntaxError
result = await fetch_data()

# ✅ CORRECT - Use await inside async function
async def main():
    result = await fetch_data()

asyncio.run(main())
```

### 1.3: Concurrency with asyncio.gather()

The power of async: run multiple tasks concurrently!

```python
import asyncio

async def task(name, duration):
    print(f"{name} started")
    await asyncio.sleep(duration)
    print(f"{name} finished")
    return f"{name} result"

async def main():
    # Run three tasks concurrently
    results = await asyncio.gather(
        task("Task A", 3),
        task("Task B", 2),
        task("Task C", 1)
    )
    print(f"All results: {results}")

start = asyncio.get_event_loop().time()
asyncio.run(main())
elapsed = asyncio.get_event_loop().time() - start
print(f"Total time: {elapsed:.1f}s")

# OUTPUT:
# Task A started
# Task B started
# Task C started
# Task C finished
# Task B finished
# Task A finished
# All results: ['Task A result', 'Task B result', 'Task C result']
# Total time: 3.0s (Not 3+2+1=6s!)
```

**Without `gather()` (sequential):**

```python
async def main():
    r1 = await task("Task A", 3)  # Wait 3s
    r2 = await task("Task B", 2)  # Wait 2s
    r3 = await task("Task C", 1)  # Wait 1s
    # Total: 6 seconds

asyncio.run(main())
```

---

## Part 2: Async Patterns and Real-World Scenarios (1.5 hours)

### 2.1: Async Context Managers (asynccontextmanager)

```python
import asyncio
from contextlib import asynccontextmanager

@asynccontextmanager
async def database_connection():
    """Async context manager for database connections"""
    print("Opening database connection...")
    connection = "DB_CONNECTION"
    try:
        yield connection
    finally:
        print("Closing database connection...")

async def main():
    async with database_connection() as conn:
        print(f"Using connection: {conn}")
        await asyncio.sleep(1)
        print("Query completed")

asyncio.run(main())

# OUTPUT:
# Opening database connection...
# Using connection: DB_CONNECTION
# Query completed
# Closing database connection...
```

This is important for FastAPI when managing database sessions!

### 2.2: Timeout Handling

```python
import asyncio

async def slow_operation():
    await asyncio.sleep(10)
    return "Done!"

async def main():
    try:
        result = await asyncio.wait_for(slow_operation(), timeout=2)
        print(result)
    except asyncio.TimeoutError:
        print("Operation timed out!")

asyncio.run(main())

# OUTPUT:
# Operation timed out!
```

### 2.3: Error Handling in Concurrent Tasks

```python
import asyncio

async def task_success():
    await asyncio.sleep(1)
    return "Success!"

async def task_failure():
    await asyncio.sleep(1)
    raise ValueError("Something went wrong!")

async def task_other():
    await asyncio.sleep(1)
    return "Other result"

async def main():
    # gather() with return_exceptions=True
    results = await asyncio.gather(
        task_success(),
        task_failure(),
        task_other(),
        return_exceptions=True  # Doesn't stop on first exception
    )
    
    for i, result in enumerate(results):
        if isinstance(result, Exception):
            print(f"Task {i} failed: {result}")
        else:
            print(f"Task {i} succeeded: {result}")

asyncio.run(main())

# OUTPUT:
# Task 0 succeeded: Success!
# Task 1 failed: Something went wrong!
# Task 2 succeeded: Other result
```

### 2.4: Creating Tasks with asyncio.create_task()

```python
import asyncio

async def background_task():
    for i in range(5):
        print(f"Background task iteration {i}")
        await asyncio.sleep(0.5)

async def main():
    # Create task without waiting for it to complete
    task = asyncio.create_task(background_task())
    
    # Do other stuff while task runs
    print("Main is doing other work...")
    await asyncio.sleep(1)
    print("Main is still doing other work...")
    
    # Wait for task to complete
    await task

asyncio.run(main())

# OUTPUT:
# Main is doing other work...
# Background task iteration 0
# Background task iteration 1
# Main is still doing other work...
# Background task iteration 2
# Background task iteration 3
# Background task iteration 4
```

### 2.5: Practical Example - Fetching Multiple APIs Concurrently

```python
import asyncio
import time

async def fetch_user(user_id):
    """Simulates fetching user from API"""
    print(f"Fetching user {user_id}...")
    await asyncio.sleep(1)  # Simulates network latency
    return {"id": user_id, "name": f"User {user_id}"}

async def fetch_posts(user_id):
    """Simulates fetching posts from API"""
    print(f"Fetching posts for user {user_id}...")
    await asyncio.sleep(1)  # Simulates network latency
    return {"user_id": user_id, "posts": [f"Post {i}" for i in range(3)]}

async def fetch_user_data(user_id):
    """Fetch both user and posts concurrently"""
    user, posts = await asyncio.gather(
        fetch_user(user_id),
        fetch_posts(user_id)
    )
    return {**user, **posts}

async def main():
    start = time.time()
    
    # Fetch data for 3 users concurrently
    results = await asyncio.gather(
        fetch_user_data(1),
        fetch_user_data(2),
        fetch_user_data(3)
    )
    
    elapsed = time.time() - start
    print(f"\nTotal time: {elapsed:.1f}s")
    print(f"Results: {results}")

asyncio.run(main())

# OUTPUT:
# Fetching user 1...
# Fetching posts for user 1...
# Fetching user 2...
# Fetching posts for user 2...
# Fetching user 3...
# Fetching posts for user 3...
#
# Total time: 2.0s (Not 6s!)
# Results: [
#   {'id': 1, 'name': 'User 1', 'user_id': 1, 'posts': [...]},
#   {'id': 2, 'name': 'User 2', 'user_id': 2, 'posts': [...]},
#   {'id': 3, 'name': 'User 3', 'user_id': 3, 'posts': [...]}
# ]
```

---

## Part 3: Event Loop and asyncio Internals (1 hour)

### 3.1: Understanding the Event Loop

```python
import asyncio

async def task1():
    print("Task 1: Starting")
    await asyncio.sleep(1)
    print("Task 1: Done")

async def task2():
    print("Task 2: Starting")
    await asyncio.sleep(0.5)
    print("Task 2: Done")

# The event loop is like a manager that:
# 1. Runs task1 until it hits 'await'
# 2. Switches to task2 until it hits 'await'
# 3. Waits for tasks to complete
# 4. Repeats until all tasks are done

async def main():
    await asyncio.gather(task1(), task2())

asyncio.run(main())

# OUTPUT (notice the interleaving):
# Task 1: Starting
# Task 2: Starting
# Task 2: Done      (Task 2 finishes first - shorter wait)
# Task 1: Done      (Task 1 finishes next)
```

### 3.2: Get Event Loop (Advanced)

```python
import asyncio

async def example():
    # Get the currently running event loop
    loop = asyncio.get_running_loop()
    print(f"Event loop: {loop}")
    
    # You can do things like:
    # - Check if loop is running
    # - Schedule callbacks
    # - Get the current time

asyncio.run(example())
```

### 3.3: Async vs Sync - CPU-bound Work

⚠️ **Important:** Async is great for I/O-bound work (network, database) but NOT for CPU-intensive tasks.

```python
import asyncio
import time

# ❌ DON'T do this with async
async def cpu_heavy():
    total = 0
    for i in range(10**8):  # 100 million iterations
        total += i
    return total

async def main():
    # Even though we use async, this blocks everything
    result1 = await cpu_heavy()
    result2 = await cpu_heavy()
    # Total: 20+ seconds - no concurrency benefit!

asyncio.run(main())

# ✅ DO use threads/processes for CPU work
from concurrent.futures import ThreadPoolExecutor

executor = ThreadPoolExecutor(max_workers=2)

async def main_better():
    loop = asyncio.get_running_loop()
    result1 = await loop.run_in_executor(executor, cpu_heavy)
    result2 = await loop.run_in_executor(executor, cpu_heavy)
    # Now these can run concurrently!
```

---

## Part 4: Why FastAPI Uses Async (0.5 hours)

### Understanding FastAPI's Async Advantage

```python
# Imagine a web server that handles 100 concurrent requests
# Each request takes 1 second for database query

# Synchronous Flask approach:
# - Handle request 1: 1 second
# - Handle request 2: 1 second
# - Handle request 3: 1 second
# ...
# - Handle request 100: 1 second
# TOTAL: 100 seconds to handle all requests!

# Asynchronous FastAPI approach:
# - Handle requests 1-100 concurrently
# - Each waits 1 second for DB
# - While 1 is waiting for DB, 2-100 can be processed
# TOTAL: ~1 second to handle all requests!

# The key: use await for I/O, allowing other requests to run
```

**FastAPI's async pattern:**

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/users/{user_id}")
async def get_user(user_id: int):
    # This is async, so while waiting for DB, other requests can be processed
    user = await database.fetch_user(user_id)
    return user
```

---

## Async/Await Exercises

### Exercise 1: Sequential vs Concurrent

Write async functions to compare timing.

```python
import asyncio

async def download(file_name, duration):
    print(f"Downloading {file_name}...")
    await asyncio.sleep(duration)
    return f"{file_name} downloaded"

async def sequential():
    """Downloads files one by one"""
    # Your code here
    pass

async def concurrent():
    """Downloads files at the same time"""
    # Your code here
    pass

# Time both approaches
```

### Exercise 2: Error Handling

Handle errors in concurrent tasks.

```python
async def api_call(url, should_fail=False):
    await asyncio.sleep(1)
    if should_fail:
        raise Exception(f"Failed to fetch {url}")
    return f"Data from {url}"

async def fetch_all():
    # Fetch from 3 URLs, 1 will fail
    # Your code here
    pass
```

---

# TYPE HINTS DEEP DIVE - 3 HOURS

## Part 1: Why Type Hints Matter (0.5 hours)

```python
# ❌ WITHOUT type hints - Confusing!
def calculate_age(person):
    # What type is person? What should we return?
    # Is it a dict? An object? A tuple?
    return person["age"]

# ✅ WITH type hints - Clear!
def calculate_age(person: dict) -> int:
    return person["age"]
```

### Benefits of Type Hints

1. **IDE Auto-completion** - VS Code knows what methods are available
2. **Type Checking** - Tools like `mypy` catch errors before runtime
3. **Self-Documentation** - Developers know what to pass and what to expect
4. **Pydantic Validation** - FastAPI uses type hints for data validation!

```python
from typing import List, Dict

# FastAPI REQUIRES type hints!
@app.post("/users/")
def create_user(name: str, age: int) -> dict:
    # FastAPI knows to:
    # - Expect 'name' as string
    # - Expect 'age' as integer
    # - Return a dictionary
    # - Automatically validate inputs!
    return {"name": name, "age": age}
```

---

## Part 2: Basic Type Hints (1 hour)

### 2.1: Simple Types

```python
# String
name: str = "Alice"

# Integer
age: int = 30

# Float
height: float = 5.8

# Boolean
is_active: bool = True

# None (special type)
value: None = None

# Lists
numbers: list = [1, 2, 3]  # Not specific about content
numbers: list[int] = [1, 2, 3]  # Specific - list of integers (Python 3.9+)
from typing import List
numbers: List[int] = [1, 2, 3]  # Older style (Python 3.8 and before)

# Dictionaries
person: dict = {"name": "Alice", "age": 30}  # Not specific
person: dict[str, str | int] = {"name": "Alice", "age": 30}  # Specific (Python 3.10+)
from typing import Dict
person: Dict[str, int] = {"age": 30}  # Keys are strings, values are integers

# Tuples
coordinates: tuple = (10, 20)  # Not specific
coordinates: tuple[int, int] = (10, 20)  # Specific - exactly two ints

# Sets
unique_ids: set[int] = {1, 2, 3}
```

### 2.2: Optional Types (Can Be None)

```python
from typing import Optional

# These are equivalent:
age: Optional[int] = None  # Can be int OR None
age: int | None = None     # Python 3.10+ syntax

# In functions
def get_user_by_id(user_id: int) -> Optional[dict]:
    # Might return a dict or None
    if user_id == 0:
        return None
    return {"id": user_id, "name": "Alice"}

# With Union (older style)
from typing import Union
age: Union[int, None] = None
```

### 2.3: Union Types (Multiple Possibilities)

```python
from typing import Union

# Python 3.10+ syntax (cleaner)
result: str | int | None = None
result = "success"
result = 404
result = None  # All valid

# Older syntax (Python 3.8-3.9)
from typing import Union
result: Union[str, int, None] = None

# Function that returns different types
def process_data(data: str | int) -> float:
    if isinstance(data, str):
        return float(len(data))
    return float(data)
```

---

## Part 3: Generic Types (1 hour)

### 3.1: Lists, Dicts, and Tuples

```python
from typing import List, Dict, Tuple, Set

# List of specific type
integers: List[int] = [1, 2, 3]
strings: List[str] = ["a", "b", "c"]

# Nested lists
matrix: List[List[int]] = [[1, 2], [3, 4]]

# Dictionary with specific types
user: Dict[str, int] = {"age": 30, "score": 100}

# Mixed dictionary (string keys, any value)
config: Dict[str, str | int | bool] = {
    "debug": True,
    "port": 8000,
    "host": "localhost"
}

# Tuple with specific lengths and types
coordinate: Tuple[int, int] = (10, 20)
rgb_color: Tuple[int, int, int] = (255, 0, 128)

# Set
unique_tags: Set[str] = {"python", "fastapi", "async"}

# Function returning a list
def get_user_ids() -> List[int]:
    return [1, 2, 3, 4, 5]

# Function taking list as argument
def process_names(names: List[str]) -> int:
    return len(names)
```

### 3.2: Callable Types

```python
from typing import Callable

# Function that takes a function as parameter
def apply_function(func: Callable[[int, int], int], a: int, b: int) -> int:
    """
    Callable[[int, int], int] means:
    - Takes two integers as arguments
    - Returns an integer
    """
    return func(a, b)

def add(a: int, b: int) -> int:
    return a + b

result = apply_function(add, 5, 3)  # Returns 8

# Decorator type hints
from typing import TypeVar, Callable
from functools import wraps

F = TypeVar('F', bound=Callable)

def my_decorator(func: F) -> F:
    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```

### 3.3: Iterables and Sequences

```python
from typing import Iterable, Sequence, Iterator

# Iterable - anything you can loop over
def process_items(items: Iterable[str]) -> None:
    for item in items:
        print(item)

# Can pass list, tuple, set, generator, etc.
process_items(["a", "b", "c"])
process_items(("a", "b", "c"))
process_items({"a", "b", "c"})

# Sequence - has length and can be indexed
def get_first(sequence: Sequence[int]) -> int:
    return sequence[0]

# Iterator - produces values one at a time
def create_iterator() -> Iterator[int]:
    for i in range(5):
        yield i
```

---

## Part 4: Type Hints for Classes and Functions (0.5 hours)

### 4.1: Class Type Hints

```python
class User:
    def __init__(self, name: str, age: int) -> None:
        self.name: str = name
        self.age: int = age
    
    def get_info(self) -> str:
        return f"{self.name} is {self.age} years old"
    
    def set_age(self, age: int) -> None:
        self.age = age

# Using the class
user: User = User("Alice", 30)
info: str = user.get_info()
```

### 4.2: Return Type Hints

```python
# No return
def greet(name: str) -> None:
    print(f"Hello, {name}")

# Returns value
def add(a: int, b: int) -> int:
    return a + b

# Returns multiple values (tuple)
def get_coordinates() -> tuple[int, int]:
    return (10, 20)

# Can destructure
x: int
y: int
x, y = get_coordinates()

# Returns optional
def find_user(user_id: int) -> dict | None:
    if user_id == 0:
        return None
    return {"id": user_id, "name": "Alice"}
```

### 4.3: Type Aliases (Simplifying Complex Types)

```python
from typing import TypeAlias

# Create an alias for complex type
UserId: TypeAlias = int
UserProfile: TypeAlias = dict[str, str | int]

def get_user(user_id: UserId) -> UserProfile:
    return {"id": user_id, "name": "Alice", "age": 30}

# Or simpler way (Python 3.9+)
UserId = int
UserProfile = dict[str, str | int]
```

---

## Part 5: Type Hints for Decorators and Async (0.5 hours)

### 5.1: Typing Decorators Properly

```python
from typing import Callable, TypeVar, cast
from functools import wraps

F = TypeVar('F', bound=Callable)

def my_decorator(func: F) -> F:
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        return func(*args, **kwargs)
    return cast(F, wrapper)

# Now your decorated function keeps its type hints
@my_decorator
def add(a: int, b: int) -> int:
    return a + b

# IDE still knows that add returns int!
result: int = add(5, 3)
```

### 5.2: Typing Async Functions

```python
import asyncio
from typing import Awaitable

# Async function returns a coroutine
async def fetch_data() -> dict:
    await asyncio.sleep(1)
    return {"data": "value"}

# Type hint for awaitable
async def process() -> None:
    result: dict = await fetch_data()
    print(result)

# Function that accepts async functions
async def run_async(async_func: Callable[[], Awaitable[dict]]) -> dict:
    return await async_func()
```

---

## Type Hints Exercises

### Exercise 1: Basic Type Hints

Add type hints to these functions:

```python
# Exercise 1
def get_full_name(first, last):
    return f"{first} {last}"

# Exercise 2
def calculate_total(items):
    return sum(items)

# Exercise 3
def filter_positive_numbers(numbers):
    return [n for n in numbers if n > 0]
```

### Exercise 2: Complex Type Hints

Add type hints:

```python
def process_users(users):
    """
    Takes list of user dicts
    Returns list of user IDs
    """
    return [user["id"] for user in users]

def create_response(status, data=None):
    """
    Takes status code and optional data
    Returns dict with status and data
    """
    return {"status": status, "data": data}
```

---

# BUILT-IN TOOLS - 2 HOURS

## Part 1: Context Managers (1 hour)

### 1.1: Understanding the `with` Statement

Without context managers (manual cleanup):

```python
# ❌ Manual resource management - error-prone
file = open("data.txt", "r")
data = file.read()
file.close()  # Must remember to close!

# If error occurs before close(), file stays open!
file = open("data.txt", "r")
data = file.read()
file.write("more data")  # Oops, opened in read mode - error!
file.close()  # Never reached!
```

With context managers (automatic cleanup):

```python
# ✅ Automatic resource management
with open("data.txt", "r") as file:
    data = file.read()
# File is automatically closed here, even if error occurs!
```

### 1.2: How Context Managers Work

```python
class MyContextManager:
    def __enter__(self):
        print("Entering...")
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        print("Exiting...")
        return False  # Don't suppress exceptions

with MyContextManager() as cm:
    print("Inside context")

# OUTPUT:
# Entering...
# Inside context
# Exiting...
```

The `__exit__` method is ALWAYS called, even if an exception occurs:

```python
class MyContextManager:
    def __enter__(self):
        print("Setup resources")
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        print("Cleanup resources")
        if exc_type is not None:
            print(f"Exception occurred: {exc_type.__name__}")
        return False  # Don't suppress exceptions

with MyContextManager():
    raise ValueError("Something wrong!")

# OUTPUT:
# Setup resources
# Cleanup resources
# Exception occurred: ValueError
# (ValueError is re-raised)
```

### 1.3: Practical Example - Database Connection

```python
class DatabaseConnection:
    def __init__(self, connection_string):
        self.connection_string = connection_string
        self.connection = None
    
    def __enter__(self):
        print(f"Connecting to {self.connection_string}")
        self.connection = f"Connection({self.connection_string})"
        return self.connection
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        print(f"Closing connection")
        self.connection = None

# Usage
with DatabaseConnection("postgres://localhost") as conn:
    print(f"Using: {conn}")
    # Simulate query
    print("Executing query")

# OUTPUT:
# Connecting to postgres://localhost
# Using: Connection(postgres://localhost)
# Executing query
# Closing connection
```

### 1.4: Using contextlib.contextmanager (Easier)

```python
from contextlib import contextmanager

@contextmanager
def database_connection(db_url):
    """Context manager for database connections"""
    print(f"Opening connection to {db_url}")
    connection = f"Connected to {db_url}"
    try:
        yield connection
    finally:
        print(f"Closing connection")

# Usage
with database_connection("postgres://localhost") as conn:
    print(f"Using: {conn}")

# OUTPUT:
# Opening connection to postgres://localhost
# Using: Connected to postgres://localhost
# Closing connection
```

**Why `try/finally`?** Because we want cleanup to happen even if exception occurs:

```python
@contextmanager
def database_connection(db_url):
    print(f"Opening connection")
    try:
        yield f"Connected"
    finally:
        print(f"Closing connection")  # Always runs!

with database_connection("postgres://localhost"):
    raise ValueError("Error during query!")

# OUTPUT:
# Opening connection
# Closing connection
# ValueError raised (after cleanup!)
```

---

## Part 2: Generators and yield (0.75 hours)

### 2.1: Understanding Generators

```python
# Regular function
def count_regular():
    result = []
    for i in range(3):
        result.append(i)
    return result

print(count_regular())  # [0, 1, 2]

# Generator function (uses yield)
def count_generator():
    for i in range(3):
        yield i

gen = count_generator()  # Returns generator object
print(next(gen))  # 0
print(next(gen))  # 1
print(next(gen))  # 2

# Loop over generator
for value in count_generator():
    print(value)  # 0, 1, 2
```

### 2.2: Generators are Memory Efficient

```python
import sys

# List - stores all values in memory
def create_list(n):
    result = []
    for i in range(n):
        result.append(i)
    return result

# Generator - generates values on-the-fly
def create_generator(n):
    for i in range(n):
        yield i

# Compare memory usage
list_1000 = create_list(1000)
print(f"List size: {sys.getsizeof(list_1000)} bytes")  # ~8024 bytes

gen_1000 = create_generator(1000)
print(f"Generator size: {sys.getsizeof(gen_1000)} bytes")  # ~112 bytes

# Generator is much smaller because values are created on demand!
```

### 2.3: Practical Example - Reading Large Files

```python
# ❌ Without generator - loads entire file in memory
def read_file_all(filename):
    with open(filename) as f:
        return f.readlines()  # Loads all lines

# ✅ With generator - reads line by line
def read_file_generator(filename):
    with open(filename) as f:
        for line in f:
            yield line.strip()

# Usage
for line in read_file_generator("large_file.txt"):
    process_line(line)
    # Only one line in memory at a time!
```

### 2.4: Generator Expressions

```python
# List comprehension - creates entire list
squares_list = [x**2 for x in range(1000)]

# Generator expression - creates values on demand
squares_gen = (x**2 for x in range(1000))

# Only difference: () instead of []
# But generator uses much less memory!

for square in squares_gen:
    print(square)
```

---

## Part 3: Comprehensions (0.5 hours)

### 3.1: List Comprehensions

```python
# Regular way
numbers = []
for x in range(5):
    numbers.append(x * 2)

# Comprehension way (cleaner)
numbers = [x * 2 for x in range(5)]
# Result: [0, 2, 4, 6, 8]

# With condition
even_numbers = [x for x in range(10) if x % 2 == 0]
# Result: [0, 2, 4, 6, 8]

# Nested
matrix = [[i + j for j in range(3)] for i in range(3)]
# Result: [[0, 1, 2], [1, 2, 3], [2, 3, 4]]
```

### 3.2: Dictionary Comprehensions

```python
# Regular way
user_ages = {}
for user in users:
    user_ages[user["id"]] = user["age"]

# Comprehension way
user_ages = {user["id"]: user["age"] for user in users}

# Example
users = [{"id": 1, "name": "Alice", "age": 30}]
user_ages = {u["id"]: u["age"] for u in users}
# Result: {1: 30}
```

### 3.3: Set Comprehensions

```python
# Create set of unique items
numbers = [1, 2, 2, 3, 3, 3, 4]
unique = {x for x in numbers}
# Result: {1, 2, 3, 4}

# With condition
even_unique = {x for x in numbers if x % 2 == 0}
# Result: {2, 4}
```

---

## Part 4: functools and itertools (0.75 hours)

### 4.1: functools.wraps (Critical for Decorators)

Already covered in Decorators section, but important reminder:

```python
from functools import wraps

def my_decorator(func):
    @wraps(func)  # Preserves function metadata
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```

### 4.2: functools.lru_cache (Memoization)

```python
from functools import lru_cache

@lru_cache(maxsize=128)  # Cache up to 128 results
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

# First call: calculates
print(fibonacci(10))  # Takes time

# Second call: returns cached result
print(fibonacci(10))  # Instant!

# Useful for expensive computations
@lru_cache(maxsize=None)  # Unlimited cache
def expensive_computation(x):
    import time
    time.sleep(1)
    return x * 2
```

### 4.3: itertools - Working with Iterables

```python
from itertools import chain, combinations, islice

# chain - combine iterables
result = list(chain([1, 2], [3, 4], [5, 6]))
# Result: [1, 2, 3, 4, 5, 6]

# combinations - all possible combinations
result = list(combinations([1, 2, 3], 2))
# Result: [(1, 2), (1, 3), (2, 3)]

# islice - slice an iterable
result = list(islice(range(10), 2, 5))
# Result: [2, 3, 4]

# zip - combine iterables
result = list(zip(['a', 'b', 'c'], [1, 2, 3]))
# Result: [('a', 1), ('b', 2), ('c', 3)]
```

---

# WEEK 1 CHECKPOINT PROJECT

## Project: Build 3 Custom Decorators

### Objective

Create three decorators that demonstrate:

1. Simple decorator with logging
2. Decorator with parameters (rate limiting)
3. Stacked decorators

### Project Requirements

#### Decorator 1: Logging Decorator

```python
# Create a decorator that logs function calls

def logger(func):
    """
    Logs function name, arguments, return value, and execution time
    
    Requirements:
    - Preserve function metadata with @wraps
    - Print function name
    - Print arguments (both positional and keyword)
    - Print return value
    - Print execution time in milliseconds
    - Handle exceptions (should still log before raising)
    """
    pass

# Test it:
@logger
def add(a, b):
    """Add two numbers"""
    return a + b

result = add(5, 3)

# Expected output:
# Calling: add
# Arguments: args=(5, 3), kwargs={}
# Returned: 8
# Execution time: 0.12ms
```

#### Decorator 2: Rate Limiter Decorator

```python
def rate_limit(calls_per_second):
    """
    Limits how many times a function can be called per second
    
    Requirements:
    - Accept parameter: calls_per_second
    - Track function calls with timestamps
    - Raise exception if limit exceeded
    - Message should include: limit, time window, when limit resets
    """
    pass

# Test it:
@rate_limit(calls_per_second=2)
def api_request():
    return "Success"

# Call it 3 times quickly
for i in range(3):
    try:
        print(api_request())
    except Exception as e:
        print(f"Error: {e}")

# Expected output:
# Success
# Success
# Error: Rate limit (2 calls/sec) exceeded. Try again in Xs
```

#### Decorator 3: Type Validator Decorator

```python
def validate_args(**type_checks):
    """
    Validates argument types before calling function
    
    Requirements:
    - Accept keyword arguments: param_name=expected_type
    - Check types before function execution
    - Raise TypeError with helpful message if type wrong
    - Support multiple argument validation
    """
    pass

# Test it:
@validate_args(name=str, age=int)
def create_user(name, age):
    return f"User: {name}, Age: {age}"

print(create_user("Alice", 30))  # Works
try:
    create_user("Bob", "thirty")  # Should fail
except TypeError as e:
    print(f"Error: {e}")

# Expected output:
# User: Alice, Age: 30
# Error: Argument 'age' must be <class 'int'>, got <class 'str'>
```

#### Decorator 4: Retry Decorator (Bonus Challenge)

```python
def retry(max_attempts=3, delay=1):
    """
    Retries function execution if it raises an exception
    
    Requirements:
    - Accept parameters: max_attempts, delay (seconds between retries)
    - Retry on exception
    - Print retry attempts
    - Re-raise exception if all attempts fail
    """
    pass

# Test it:
attempt_count = 0

@retry(max_attempts=3, delay=0.5)
def unstable_function():
    global attempt_count
    attempt_count += 1
    if attempt_count < 3:
        raise ValueError("Temporary error")
    return "Success!"

result = unstable_function()
print(result)

# Expected output:
# Attempt 1/3
# Attempt 2/3 (retrying after 0.5s)
# Attempt 3/3
# Success!
```

### Deliverables

Create a Python file `week_1_decorators.py` with:

1. All three decorators implemented
2. Test cases for each decorator
3. Comments explaining how each works
4. Bonus: The retry decorator

### How to Submit

1. Create the file
2. Run all tests to ensure they work
3. Save to `/mnt/user-data/outputs/week_1_decorators.py`

---

# EXERCISE SOLUTIONS

## Decorators Exercise Solutions

### Exercise 1: Logger Decorator

```python
import time
from functools import wraps

def logger(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling: {func.__name__}")
        print(f"Arguments: args={args}, kwargs={kwargs}")
        
        start = time.time()
        result = func(*args, **kwargs)
        elapsed = (time.time() - start) * 1000  # Convert to ms
        
        print(f"Returned: {result}")
        print(f"Execution time: {elapsed:.2f}ms")
        return result
    return wrapper

@logger
def add(a, b):
    """Add two numbers"""
    time.sleep(0.001)
    return a + b

add(5, 3)
```

### Exercise 2: Cache Decorator

```python
from functools import wraps

def cache_decorator(func):
    cache = {}
    
    @wraps(func)
    def wrapper(*args, **kwargs):
        # Create cache key from arguments
        key = str(args) + str(kwargs)
        
        if key in cache:
            print(f"Cache hit for {func.__name__}{args}")
            return cache[key]
        
        print(f"Cache miss for {func.__name__}{args}, computing...")
        result = func(*args, **kwargs)
        cache[key] = result
        return result
    
    return wrapper

@cache_decorator
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

print(fibonacci(5))  # Computes
print(fibonacci(5))  # From cache
```

### Exercise 3: Type Validator

```python
from functools import wraps

def validate_types(**types):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            # Get function signature
            import inspect
            sig = inspect.signature(func)
            params = list(sig.parameters.keys())
            
            # Validate positional arguments
            for i, arg in enumerate(args):
                if i < len(params):
                    param_name = params[i]
                    if param_name in types:
                        expected_type = types[param_name]
                        if not isinstance(arg, expected_type):
                            raise TypeError(
                                f"Argument '{param_name}' must be {expected_type}, "
                                f"got {type(arg)}"
                            )
            
            # Validate keyword arguments
            for param_name, value in kwargs.items():
                if param_name in types:
                    expected_type = types[param_name]
                    if not isinstance(value, expected_type):
                        raise TypeError(
                            f"Argument '{param_name}' must be {expected_type}, "
                            f"got {type(value)}"
                        )
            
            return func(*args, **kwargs)
        return wrapper
    return decorator

@validate_types(a=int, b=int)
def multiply(a, b):
    return a * b

print(multiply(3, 4))  # Works
# multiply("3", "4")  # Raises TypeError
```

## Async/Await Exercise Solutions

### Exercise 1: Sequential vs Concurrent

```python
import asyncio
import time

async def download(file_name, duration):
    print(f"Downloading {file_name}...")
    await asyncio.sleep(duration)
    return f"{file_name} downloaded"

async def sequential():
    """Downloads files one by one"""
    start = time.time()
    r1 = await download("file1.txt", 2)
    r2 = await download("file2.txt", 2)
    r3 = await download("file3.txt", 2)
    elapsed = time.time() - start
    print(f"Sequential took: {elapsed:.1f}s")
    return [r1, r2, r3]

async def concurrent():
    """Downloads files at the same time"""
    start = time.time()
    results = await asyncio.gather(
        download("file1.txt", 2),
        download("file2.txt", 2),
        download("file3.txt", 2)
    )
    elapsed = time.time() - start
    print(f"Concurrent took: {elapsed:.1f}s")
    return results

# Test both
print("Sequential:")
asyncio.run(sequential())

print("\nConcurrent:")
asyncio.run(concurrent())
```

### Exercise 2: Error Handling

```python
import asyncio

async def api_call(url, should_fail=False):
    await asyncio.sleep(1)
    if should_fail:
        raise Exception(f"Failed to fetch {url}")
    return f"Data from {url}"

async def fetch_all():
    """Fetch from 3 URLs, 1 will fail"""
    results = await asyncio.gather(
        api_call("http://api1.com"),
        api_call("http://api2.com", should_fail=True),
        api_call("http://api3.com"),
        return_exceptions=True  # Don't stop on exception
    )
    
    for i, result in enumerate(results):
        if isinstance(result, Exception):
            print(f"URL {i+1}: Error - {result}")
        else:
            print(f"URL {i+1}: {result}")

asyncio.run(fetch_all())
```

## Type Hints Exercise Solutions

### Exercise 1: Basic Type Hints

```python
def get_full_name(first: str, last: str) -> str:
    return f"{first} {last}"

def calculate_total(items: list[int]) -> int:
    return sum(items)

def filter_positive_numbers(numbers: list[int]) -> list[int]:
    return [n for n in numbers if n > 0]
```

### Exercise 2: Complex Type Hints

```python
from typing import Optional, Dict, List

def process_users(users: List[Dict[str, int]]) -> List[int]:
    """
    Takes list of user dicts
    Returns list of user IDs
    """
    return [user["id"] for user in users]

def create_response(
    status: int, 
    data: Optional[Dict[str, str]] = None
) -> Dict[str, object]:
    """
    Takes status code and optional data
    Returns dict with status and data
    """
    return {"status": status, "data": data}
```

---

# SUMMARY AND CHECKLIST

## Week 1 Learning Path Summary

**Total Time: 12-13 Hours**

### Decorators (3 hours)

- ✅ Understand what decorators are and why they're useful
- ✅ Write simple decorators with @wraps
- ✅ Create decorators with parameters
- ✅ Stack multiple decorators
- ✅ Preserve function metadata
- ✅ Real-world FastAPI patterns

### Async/Await (4 hours)

- ✅ Understand sync vs async
- ✅ Write async functions
- ✅ Use await keyword
- ✅ Run concurrent tasks with asyncio.gather()
- ✅ Handle errors in concurrent tasks
- ✅ Understand event loop basics
- ✅ Know when to use async (I/O-bound, not CPU-bound)

### Type Hints (3 hours)

- ✅ Basic type hints (str, int, bool, etc.)
- ✅ Container types (List, Dict, Tuple)
- ✅ Optional types (Optional, Union)
- ✅ Callable types
- ✅ Type aliases
- ✅ Type hints for decorators and async

### Built-in Tools (2 hours)

- ✅ Context managers (with statement)
- ✅ Generators (yield keyword)
- ✅ Comprehensions (list, dict, set)
- ✅ functools (wraps, lru_cache)
- ✅ itertools basics

## Week 1 Success Criteria

By end of Week 1, you should be able to:

1. **Write decorators from memory** - Simple, with parameters, stacked
2. **Explain async/await** - Why it's faster, when to use it
3. **Use type hints effectively** - For functions, classes, complex types
4. **Use context managers** - For resource management
5. **Understand generators** - Yield, memory efficiency
6. **Know FastAPI patterns** - Why these tools matter for FastAPI

## Next Steps

After Week 1, you'll have all the Python foundation needed. Week 2 starts FastAPI itself!

**Don't move to Week 2 until you can:**

- [ ] Create any decorator without looking at examples
- [ ] Explain how async/await works
- [ ] Add type hints to any Python function
- [ ] Complete the checkpoint project (3 decorators)

---

# ADDITIONAL RESOURCES

## Video Resources

- **Python Decorators**: https://www.youtube.com/watch?v=s7dgUodNn98
- **Async/Await Explained**: https://www.youtube.com/watch?v=gsE4_E6lZFo
- **Type Hints in Python**: https://www.youtube.com/watch?v=QOrJrT2_tzU

## Documentation

- Python Official Docs: https://docs.python.org/3/
- Real Python Tutorials: https://realpython.com/
- FastAPI Docs (reference): https://fastapi.tiangolo.com/

## Practice Platforms

- LeetCode (functions and data structures)
- HackerRank (Python challenges)
- Codewars (decorator challenges)

---

**End of Week 1 Content**

Good luck! This foundation is critical for FastAPI mastery. 🚀