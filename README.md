# Snippets

Self-contained copy-and-paste code snippets.

1. [Cache Function Outputs](#cache-function-outputs)
1. [Time a Function](#time-a-function)
1. [Install Google Chrome](#install-google-chrome)
1. [Google Colab Terminal](#google-colab-terminal)

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
            time_string = f"{elapsed_time:.2f}s"
        else:
            time_string = f"{hours}h {minutes}m {seconds}s"
        print(f"{func.__name__} executed in {time_string}")
        return result
    return wrapper
```

## Google Colab Terminal

```python
%pip install --quiet --progress-bar off colab-xterm
%reload_ext colabxterm
%xterm
```

## Install Google Chrome

```python
def install_chrome(version: str = "current"):
    import requests, shutil, subprocess, tempfile
    with tempfile.TemporaryDirectory() as temp_dir:
        install_path = f"{temp_dir}/google-chrome.deb"
        url = f"https://dl.google.com/linux/direct/google-chrome-stable_{version}_amd64.deb"
        response = requests.get(url, stream=True)
        response.raise_for_status()
        with open(install_path, mode="wb") as file:
            for chunk in response.iter_content(chunk_size=8192):
                file.write(chunk)
        result = subprocess.run(f"dpkg --install {install_path}", shell=True)
        if result.returncode != 0:
            subprocess.run("sudo apt-get --yes --fix-broken install", shell=True, check=True)
        print(shutil.which("google-chrome"))
        print(subprocess.check_output("google-chrome --version", shell=True, text=True))

install_chrome()
```

## Install ChromeDriver

```python
def install_chromedriver(version: str = None):
    import os, requests, shutil, subprocess, tempfile, zipfile
    chromedriver_path = "/usr/bin/chromedriver"
    url = "https://chromedriver.storage.googleapis.com"
    if not version:
        response = requests.get(f"{url}/LATEST_RELEASE")
        response.raise_for_status()
        version = response.text
    with tempfile.TemporaryDirectory() as temp_dir:
        install_path = f"/{temp_dir}/chromedriver_linux64.zip"
        response = requests.get(f"{url}/{version}/chromedriver_linux64.zip")
        response.raise_for_status()
        with open(install_path, mode="wb") as file:
            file.write(response.content)
        with zipfile.ZipFile(install_path, mode="r") as zip_ref:
            zip_ref.extractall(path="/usr/bin")
        os.chmod(chromedriver_path, mode=0o755)
        print(shutil.which("chromedriver"))
        print(subprocess.check_output(f"{chromedriver_path} --version", shell=True, text=True))

install_chromedriver()
```


