# Decorators

## @wraps

**What it is:**
-

**Why it matters:**
Without `@wraps`, a decorated function loses its original `__name__` and docstring — breaks debugging and introspection.

**Example:**
```python
from functools import wraps

def timer(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        import time
        start = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__} took {time.time() - start:.4f}s")
        return result
    return wrapper
```

**Gotchas:**
-

---

## @retry (custom)

**What it is:**
-

**Example:**
```python

```
