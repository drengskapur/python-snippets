# Snippets

Self-contained copy-and-paste code snippets.

1. [Cache Function Outputs](#cache-function-outputs)
2. [Time a Function](#time-a-function)
3. [Install Google Chrome](#install-google-chrome)

## Cache Function Outputs

```python
def cacheit(maxsize=None, verbose=False):
    import functools
    def cache_decorator(func):
        cached_func = functools.lru_cache(maxsize=maxsize)(func)
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            before_misses = cached_func.cache_info().misses
            result = cached_func(*args, **kwargs)
            if verbose and cached_func.cache_info().misses == before_misses:
                print(f"HIT! {func.__name__}")
            elif verbose:
                print(f"MISS! {func.__name__}")
            return result
        return wrapper
    return cache_decorator
```

## Time a Function

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

## Install Google Chrome

```python
def install_chrome(version: str = "current"):
    import requests, subprocess
    url = f"https://dl.google.com/linux/direct/google-chrome-stable_{version}_amd64.deb"
    with open("/tmp/google-chrome.deb", "wb") as file:
        [file.write(chunk) for chunk in requests.get(url, stream=True).iter_content(chunk_size=8192)]
    if subprocess.run(f"dpkg --install /tmp/google-chrome.deb", shell=True).returncode != 0:
        subprocess.run("sudo apt-get --yes --fix-broken install", shell=True, check=True)
    print(subprocess.check_output("google-chrome --version", shell=True, text=True))

install_chrome("current")
```
