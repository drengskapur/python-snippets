# Python Snippets

Self-contained copy-and-paste Python code snippets.

Utilities

1. [Git LFS Track Files Over 100MB](#git-lfs-track-files-over-100mb)
1. [Cache Function Outputs](#cache-function-outputs)
1. [Time a Function](#time-a-function) 
1. [Checksum Utilities](#checksum-utilities)

Installers

1. [Install Google Chrome](#install-google-chrome)
1. [Install ChromeDriver](#install-chromedriver)
1. [Install Firefox](#install-firefox)
1. [Install GeckoDriver](#install-geckodriver) 
1. [Install VS Code Server](#install-vs-code-server)
1. [Install Node](#install-node)

Google Colab

1. [Google Colab Terminal](#google-colab-terminal)
1. [Create a Folder Shortcut to Google Drive Folder in the File Browser](#create-a-folder-shortcut-to-google-drive-folder-in-the-file-browser)
1. [Alias](#alias)  

AutoML

1. [Install autogluon on Google Colab](#install-autogluon-on-google-colab)
1. [Install auto-sklearn 0.15.0  on Google Colab](#install-auto-sklearn-0150-on-google-colab)
1. [Install tpot on Google Colab](#install-tpot-0122-on-google-colab) 

## Git LFS Track Files Over 100MB

```python
#!/usr/bin/env python3
import subprocess
from pathlib import Path

SKIP_LIST = [
    ".git",
    "build",
    "venv",
    ".venv",
    "node_modules",
]

SIZE_THRESHOLD_IN_MB = 100
CALCULATED_THRESHOLD = SIZE_THRESHOLD_IN_MB * 1024 * 1024


def track_large_files(directory):
    try:
        for item in directory.iterdir():
            if item.name in SKIP_LIST or item.is_symlink():
                continue

            if item.is_dir():
                track_large_files(item)
            elif item.is_file():
                try:
                    file_size = item.stat().st_size
                except FileNotFoundError:
                    print(f"File not found: {item}")
                    continue
                except PermissionError:
                    print(f"Permission denied: {item}")
                    continue

                if file_size > CALCULATED_THRESHOLD:
                    try:
                        subprocess.run(["git", "lfs", "track", str(item)], check=True)
                        print(f"Tracked file: {item}")
                    except subprocess.CalledProcessError as e:
                        print(f"Error tracking file: {item}")
                        print(f"Error message: {e}")
    except PermissionError:
        print(f"Permission denied: {directory}")
    except FileNotFoundError:
        print(f"Directory not found: {directory}")


def install_git_lfs():
    try:
        subprocess.run(["git", "lfs", "install"], check=True)
    except subprocess.CalledProcessError as e:
        print("Error installing Git LFS")
        print(f"Error message: {e}")
        exit(1)


directory = Path(".")
install_git_lfs()
track_large_files(directory)
```

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

## Checksum Utilities

```python
def calculate_checksum(filepath: str, hash_func):
    with open(filepath, "rb") as f:
        while chunk := f.read(8192):
            hash_func.update(chunk)
    return hash_func.hexdigest()

def create_checksum_file(filepaths: List[str], checksum_filename:str, hash_func):
    with open(checksum_filename, "w") as checksum_file:
        for filepath in filepaths:
            checksum = calculate_checksum(filepath, hash_func)
            checksum_line = f"{checksum} {filepath}\n"
            print(checksum_line)
            checksum_file.write(checksum_line)

    print(checksum_filename)
    return checksum_filename

def verify_checksums(checksum_filename):
    result = True
    with open(checksum_filename, "r") as checksum_file:
        import hashlib
        hash_func = hashlib.sha256()
        for line in checksum_file:
            stored_checksum, filepath = line.strip().split()
            current_checksum = calculate_checksum(filepath, hash_func)
            if current_checksum != stored_checksum:
                print(f"ERROR: CHECKSUM DOES NOT MATCH FOR: {filepath}")
                result = False
            else:
                print(f"CHECKSUM VERIFIED FOR: {filepath}")

    return result
```

## Install Google Chrome

```python
def install_chrome(version=None):
    import os, pathlib, platform, shutil, subprocess, requests, tempfile, zipfile
    platform_config = {
        ("linux", "x86_64"): {
            "platform_key": "linux64",
            "extracted_path_relative": "chrome-linux64/chrome",
            "chrome_file_path": "/usr/local/bin/chrome",
        },
        ("darwin", "arm64"): {
            "platform_key": "mac-arm64",
            "extracted_path_relative": "chrome-mac-arm64/Google Chrome for Testing.app/Contents/MacOS/Google Chrome for Testing",
            "chrome_file_path": "/Applications/Google Chrome.app/Contents/MacOS/Google Chrome",
        },
        ("darwin", "x86_64"): {
            "platform_key": "mac-x64",
            "extracted_path_relative": "chrome-mac-x64/Google Chrome for Testing.app/Contents/MacOS/Google Chrome for Testing",
            "chrome_file_path": "/Applications/Google Chrome.app/Contents/MacOS/Google Chrome",
        },
        ("windows", "AMD64"): {
            "platform_key": "win64",
            "extracted_path_relative": "chrome-win64/chrome.exe",
            "chrome_file_path": os.path.join(os.environ.get("PROGRAMFILES", "C:\\Program Files"), "Google", "Chrome", "Application", "chrome.exe"),
        },
        ("windows", "x86"): {
            "platform_key": "win32",
            "extracted_path_relative": "chrome-win32/chrome.exe",
            "chrome_file_path": os.path.join(os.environ.get("PROGRAMFILES(x86)", "C:\\Program Files (x86)"), "Google", "Chrome", "Application", "chrome.exe"),
        },
    }
    os_name = platform.system().lower()
    arch = platform.machine()
    config = platform_config.get((os_name, arch))
    if config is None:
        raise ValueError(f"UNSUPPORTED PLATFORM OR ARCHITECTURE: {os_name} {arch}")

    # FETCH DOWNLOAD URL
    json_url = "https://googlechromelabs.github.io/chrome-for-testing/latest-versions-per-milestone-with-downloads.json"
    response = requests.get(json_url)
    response.raise_for_status()
    data = response.json()

    # FETCH LATEST VERSION IF NO VERSION SPECIFIED ELSE FETCH SPECIFIED VERSION
    version = version.split(".")[0] if version else max(data["milestones"].keys(), key=int)
    downloads = data["milestones"][version]["downloads"]["chrome"]
    download_url = next(download["url"] for download in downloads if download["platform"] == config["platform_key"])

    # DOWNLOAD CHROME ZIP FILE
    with tempfile.TemporaryDirectory() as temp_dir:
        temp_dir = pathlib.Path(temp_dir)
        chrome_zip = temp_dir / f"chrome-{config['platform_key']}.zip"
        with requests.get(download_url, stream=True) as r:
            r.raise_for_status()
            with open(chrome_zip, "wb") as f:
                for chunk in r.iter_content(chunk_size=8192):
                    f.write(chunk)

        with zipfile.ZipFile(chrome_zip, "r") as zip_ref:
            zip_ref.extractall(temp_dir)
        extracted_path_absolute = temp_dir / config["extracted_path_relative"]

        # ENSURE DESTINATION PATH
        chrome_file_path = pathlib.Path(config["chrome_file_path"])
        if chrome_file_path.exists():
            chrome_file_path.unlink()
        chrome_file_path.parent.mkdir(parents=True, exist_ok=True)

        # MOVE EXECUTABLE TO DESTINATION PATH
        shutil.move(str(extracted_path_absolute), str(chrome_file_path))

        # SET EXECUTE FLAG FOR LINUX MACHINES
        if os_name == "linux":
            subprocess.run(["chmod", "+x", str(chrome_file_path)], check=True)

        print(chrome_file_path)
        print(subprocess.check_output(f"{chrome_file_path} --version", shell=True).decode().strip())

install_chrome()
```

## Install ChromeDriver

```python
def install_chromedriver(version=None):
    import os, pathlib, platform, shutil, subprocess, requests, tempfile, zipfile
    platform_config = {
        ("linux", "x86_64"): {
            "platform_key": "linux64",
            "extracted_path_relative": "chromedriver-linux64/chromedriver",
            "chromedriver_file_path": "/usr/local/bin/chromedriver"
        },
        ("darwin", "arm64"): {
            "platform_key": "mac-arm64",
            "extracted_path_relative": "chromedriver-mac-arm64/chromedriver",
            "chromedriver_file_path": "/usr/local/bin/chromedriver"
        },
        ("darwin", "x86_64"): {
            "platform_key": "mac-x64",
            "extracted_path_relative": "chromedriver-mac-x64/chromedriver",
            "chromedriver_file_path": "/usr/local/bin/chromedriver"
        },
        ("windows", "AMD64"): {
            "platform_key": "win64",
            "extracted_path_relative": "chromedriver-win64/chromedriver.exe",
            "chromedriver_file_path": "C:\\Windows\\chromedriver.exe"
        },
        ("windows", "x86"): {
            "platform_key": "win32",
            "extracted_path_relative": "chromedriver-win32/chromedriver.exe",
            "chromedriver_file_path": "C:\\Windows\\chromedriver.exe"
        }
    }
    os_name = platform.system().lower()
    arch = platform.machine()
    config = platform_config.get((os_name, arch))
    if not config:
        raise ValueError(f"UNSUPPORTED PLATFORM OR ARCHITECTURE: {os_name} {arch}")

    # FETCH DOWNLOAD URL
    json_url = "https://googlechromelabs.github.io/chrome-for-testing/latest-versions-per-milestone-with-downloads.json"
    response = requests.get(json_url)
    response.raise_for_status()
    data = response.json()
    # FETCH LATEST VERSION IF NO VERSION SPECIFIED ELSE FETCH SPECIFIED VERSION
    version = version.split(".")[0] if version else max(data["milestones"].keys(), key=int)
    downloads = data["milestones"][version]["downloads"]["chromedriver"]
    download_url = next(download["url"] for download in downloads if download["platform"] == config["platform_key"])

    # DOWNLOAD CHROMEDRIVER ZIP FILE
    with tempfile.TemporaryDirectory() as temp_dir:
        temp_dir = pathlib.Path(temp_dir)
        chromedriver_zip = temp_dir / f"chromedriver-{config['platform_key']}.zip"
        with requests.get(download_url, stream=True) as r:
            r.raise_for_status()
            with open(chromedriver_zip, "wb") as f:
                for chunk in r.iter_content(chunk_size=8192):
                    f.write(chunk)

        with zipfile.ZipFile(chromedriver_zip, "r") as zip_ref:
            zip_ref.extractall(temp_dir)
        extracted_path_absolute = temp_dir / config["extracted_path_relative"]

        # ENSURE DESTINATION PATH
        chromedriver_file_path = pathlib.Path(config["chromedriver_file_path"])
        if chromedriver_file_path.exists():
            chromedriver_file_path.unlink()
        chromedriver_file_path.parent.mkdir(parents=True, exist_ok=True)

        # MOVE EXECUTABLE TO DESTINATION PATH
        shutil.move(str(extracted_path_absolute), str(chromedriver_file_path))

        # SET EXECUTE FLAG FOR LINUX MACHINES
        if os_name == "linux":
            subprocess.run(["chmod", "+x", str(chromedriver_file_path)], check=True)

        print(chromedriver_file_path)
        print(subprocess.check_output(f"{chromedriver_file_path} --version", shell=True).decode().strip())

install_chromedriver()
```

## Install Firefox

```python
def install_firefox():
    import shutil, subprocess
    subprocess.run(["sudo", "add-apt-repository", "ppa:mozillateam/ppa", "--yes"], check=True)
    with open("/etc/apt/preferences.d/mozilla-firefox", mode="w") as file:
        file.write("Package: *\nPin: release o=LP-PPA-mozillateam\nPin-Priority: 1001")
    with open("/etc/apt/apt.conf.d/51unattended-upgrades-firefox", "w") as file:
        file.write('Unattended-Upgrade::Allowed-Origins:: "LP-PPA-mozillateam:${distro_codename}";')
    subprocess.run(["sudo", "apt-get", "update"], check=True)
    subprocess.run(["sudo", "apt-get", "--yes", "install", "firefox"], check=True)
    print(shutil.which("firefox"))
    print(subprocess.check_output(["/usr/bin/firefox", "--version"], text=True))

install_firefox()
```

## Install GeckoDriver

```python
def install_geckodriver():
    import os, requests, shutil, subprocess, tarfile, tempfile
    api_url = "https://api.github.com/repos/mozilla/geckodriver/releases/latest"
    destination_path = "/usr/bin/geckodriver"
    download_url = next(
        (asset["browser_download_url"] for asset in requests.get(api_url).json()["assets"]
         if "linux64" in asset["name"] and asset["name"].endswith(".tar.gz")), None)
    if not download_url:
        raise Exception("Could not find a Linux 64-bit geckodriver release.")
    with tempfile.TemporaryDirectory() as temp_dir, requests.get(download_url, stream=True) as response:
        response.raise_for_status()
        tar_path = os.path.join(temp_dir, "geckodriver.tar.gz")
        with open(tar_path, "wb") as file:
            shutil.copyfileobj(response.raw, file)
        with tarfile.open(tar_path, "r:gz") as tf:
            tf.extractall(path=temp_dir)
        shutil.move(os.path.join(temp_dir, "geckodriver"), destination_path)
        os.chmod(destination_path, 0o755)
    print(shutil.which("geckodriver"))
    print(subprocess.check_output(["geckodriver", "--version"], text=True).split('\n', 1)[0])

install_geckodriver()
```

## Install VS Code Server

```python
def install_vscode_server():
    import pathlib, platform, shutil, subprocess, tarfile, requests

    with requests.Session() as session:
        # GET LATEST RELEASE TAG
        response = session.get(f"https://api.github.com/repos/microsoft/vscode/releases/latest")
        response.raise_for_status()
        tag = response.json()["tag_name"]

        # GET CODE SERVER TAG COMMIT SHA
        response = session.get(f"https://api.github.com/repos/microsoft/vscode/git/ref/tags/{tag}")
        response.raise_for_status()
        tag_data = response.json()
        commit_sha = tag_data["object"]["sha"]
        sha_type = tag_data["object"]["type"]

        # GET COMMIT SHA WHEN TAG COMMIT SHA IS NOT A COMMIT
        if sha_type != "commit":
            response = session.get(f"https://api.github.com/repos/microsoft/vscode/git/tags/{commit_sha}")
            response.raise_for_status()
            commit_sha = response.json()["object"]["sha"]

        # DOWNLOAD CODE SERVER
        arch_map = {"aarch64": "arm64", "x86_64": "x64", "armv7l": "armhf"}
        arch = arch_map.get(platform.machine(), "x64")
        url = f"https://update.code.visualstudio.com/commit:{commit_sha}/server-linux-{arch}/stable"
        vscode_server_tarball = pathlib.Path.home() / f"{commit_sha}.tar.gz"
        with session.get(url, stream=True) as response:
            response.raise_for_status()
            with open(vscode_server_tarball, "wb") as file:
                shutil.copyfileobj(response.raw, file)

        # EXTRACT CODE SERVER
        vscode_server_path = pathlib.Path.home() / f".vscode-server/bin/{commit_sha}"
        if vscode_server_path.exists():
            shutil.rmtree(vscode_server_path)
        vscode_server_path.mkdir(parents=True, exist_ok=True)
        with tarfile.open(vscode_server_tarball) as tar:
            tar.extractall(path=vscode_server_path)
        vscode_server_tarball.unlink()

        # SET UP A GLOBAL CODE SERVER SYMLINK
        vscode_server_executable = vscode_server_path / f"vscode-server-linux-{arch}" / "bin" / "code-server"
        symlink_target = pathlib.Path("/usr/bin/code-server")
        if symlink_target.exists():
            symlink_target.unlink()
        symlink_target.symlink_to(vscode_server_executable)

        print(shutil.which("code-server"))
        print(subprocess.check_output([str(symlink_target), "--version"], text=True))


install_vscode_server()
```

## Install Node

```python
def install_node():
    import os
    
    # REMOVE CONFLICTING INSTALLATIONS
    !rm -rf "$(which node)"

    # INSTALL VOLTA (JAVASCRIPT TOOL MANAGER)
    if not os.path.exists("~/.volta/bin"):
        !curl -fsSL https://get.volta.sh | sudo -E bash -
        !ln -sf ~/.volta/bin/volta /usr/local/bin/volta

    # INSTALL NODE PACKAGES
    for package in ["node", "npm", "npx", "yarn"]:
        if not os.path.exists(f"~/.volta/bin/{package}"):
            !volta install {package}
            !ln -sf ~/.volta/bin/{package} /usr/local/bin/{package}

install_node()
```

## Google Colab Terminal

```python
%pip install --quiet --progress-bar off colab-xterm
%reload_ext colabxterm
%xterm
```

## Create a Folder Shortcut to Google Drive Folder in the File Browser

```python
def setup_drive_folder(google_drive_folder):
    import contextlib, google.colab, os, pathlib
    if not google_drive_folder:
        google_drive_folder = "temp"
    with contextlib.redirect_stdout(open(os.devnull, 'w')):
        google.colab.drive.mount("/content/drive", force_remount=True)
    drive_path = pathlib.Path("/content/drive/MyDrive")
    colab_notebooks_path = drive_path / "Colab Notebooks"
    project_path = colab_notebooks_path / google_drive_folder
    project_path.mkdir(parents=True, exist_ok=True)
    shortcut = pathlib.Path(f"/content/{google_drive_folder}")
    shortcut.parent.mkdir(parents=True, exist_ok=True)
    if not shortcut.exists():
        shortcut.symlink_to(project_path)
    print(f"SHORTCUT: {shortcut} --> {project_path}")
    return str(shortcut)

google_drive_folder = "temp"  # @param { type: "string" }
SHORTCUT = setup_drive_folder(google_drive_folder)
```

## Alias

```text
%alias bracket echo "Input in brackets: <%l>"
%bracket I'm in brackets!
```

<img src="https://github.com/drengskapur/python-snippets/assets/5193877/61386202-e21d-4bba-ae2c-8d4fd83e2026" width="400">

## Install autogluon on Google Colab

```python
# ---------------------------------------------------------------------------- #
# @title Install autogluon on Google Colab { display-mode: "form" }
# ---------------------------------------------------------------------------- #
%pip install uv
!git clone https://github.com/autogluon/autogluon.git
!uv pip install --system -e /content/autogluon/common/
!uv pip install --system -e /content/autogluon/core/[all]
!uv pip install --system -e /content/autogluon/features/
!uv pip install --system -e /content/autogluon/tabular/[all]
!uv pip install --system -e /content/autogluon/multimodal/
!uv pip install --system -e /content/autogluon/timeseries/[all]
!uv pip install --system -e /content/autogluon/eda/
!uv pip install --system -e /content/autogluon/autogluon/
import IPython; IPython.display.clear_output()
%pip list|grep autogluon|tr -s ' '
# ---------------------------------------------------------------------------- #
```

## Install auto-sklearn 0.15.0 on Google Colab

```python
# ---------------------------------------------------------------------------- #
# @title Install auto-sklearn 0.15.0 on Google Colab { display-mode: "form" }
# ---------------------------------------------------------------------------- #
%pip install uv
!uv pip install --system Cython==0.29.36 scipy==1.9 pyparsing==2.4
!uv pip install --system --no-build-isolation scikit-learn==0.24.2
!uv pip install --system auto-sklearn==0.15.0
import IPython; IPython.display.clear_output()
!uv pip list --system|grep auto-sklearn|tr -s ' '
# ---------------------------------------------------------------------------- #
```

## Install tpot on Google Colab

```python
# ---------------------------------------------------------------------------- #
# @title Install tpot on Google Colab { display-mode: "form" }
# ---------------------------------------------------------------------------- #
%pip install uv
!uv pip install --system -r https://raw.githubusercontent.com/EpistasisLab/tpot/master/requirements.txt
!uv pip install --system tpot
import IPython; IPython.display.clear_output()
!uv pip list --system|grep tpot|tr -s ' '
# ---------------------------------------------------------------------------- #
```


