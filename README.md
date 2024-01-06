# Snippets

Self-contained copy-and-paste code snippets.

1. [Cache Function Outputs](#cache-function-outputs)
2. [Time a Function](#time-a-function)

## Cache Function Outputs<a id="cache-function-outputs"></a>

```python
def cacheit(maxsize=None, verbose=False):
    import functools
    def cache_decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            before_misses = wrapper.cache_info().misses
            result = wrapper.cached_func(*args, **kwargs)
            if verbose and wrapper.cache_info().misses == before_misses:
                print(f"HIT! {func.__name__}")
            elif verbose:
                print(f"MISS! {func.__name__}")
            return result
        wrapper.cached_func = functools.lru_cache(maxsize=maxsize)(func)
        return wrapper
    return cache_decorator
```

## Time a Function<a id="time-a-function"></a>

```python
def timeit(func):
    import functools, time
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start_time = time.time()
        result = func(*args, **kwargs)
        end_time = time.time()
        elapsed_time = end_time - start_time
        hours = f"{int(elapsed_time // 3600)}"
        minutes = f"{int((elapsed_time % 3600) // 60)}"
        seconds = f"{int(elapsed_time % 60)}"
        if elapsed_time < 60:
            time_string = f"{elapsed_time:.2f}s
        else:
            time_string = f"{hours}h {minutes}m {seconds}s"
        print(f"{func.__name__} executed in {time_string}")
        return result
    return wrapper
```
