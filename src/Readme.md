# FastAPI + Docker + Devcontainer Explained

This document explains your development setup with FastAPI inside a Docker container, including CORS, debugging with debugpy, and network concepts.

---

## 1️⃣ Dockerfile.dev Explanation

```dockerfile
FROM python:3.12-slim

RUN apt-get update && apt-get install -y curl git vim net-tools build-essential

WORKDIR /code

ENV PYTHONPATH=/code/src
```

**Line-by-line explanation:**

- **`FROM python:3.12-slim`**  
  Uses the official Python 3.12 slim image. Lightweight, includes Python + pip, based on Debian slim.

- **`RUN apt-get update && apt-get install -y ...`**  
  Installs essential development tools:
  - `curl` → download files / test APIs  
  - `git` → version control  
  - `vim` → terminal editor  
  - `net-tools` → networking commands (`ifconfig`, `netstat`)  
  - `build-essential` → compilers for Python packages with C extensions  

- **`WORKDIR /code`**  
  Sets `/code` as the working directory inside the container.

- **`ENV PYTHONPATH=/code/src`**  
  Adds `/code/src` to Python’s module search path so you can import project modules directly.

**Purpose:** Creates a dev-friendly Python environment for FastAPI with essential tools and proper Python path.

---

## 2️⃣ FastAPI Application Explanation

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from . import smokeTest

import debugpy

debugpy.listen(("0.0.0.0", 5678))
# debugpy.wait_for_client()

app = FastAPI()

origins = [
    "*"
]

app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

app.include_router(smokeTest.router, prefix="/smoke-test")

@app.get('/')
def health():
    return {
        "message": "OK"
    }
```

**Explanation:**

### Imports
- `FastAPI` → main app class  
- `CORSMiddleware` → handles cross-origin requests  
- `.smokeTest` → local module with additional API routes  
- `debugpy` → Python debugger for VS Code

### Debugging
```python
debugpy.listen(("0.0.0.0", 5678))
# debugpy.wait_for_client()
```
- Opens a debug server inside the container.  
- `0.0.0.0` → listens on all interfaces (needed for Docker host to connect).  
- `5678` → port number VS Code attaches to.  
- `wait_for_client()` → optional, pauses app until debugger attaches.

### FastAPI App
- `app = FastAPI()` → initializes the app  
- CORS setup allows any origin to call the API, all methods, headers, and credentials  
- `app.include_router(smokeTest.router, prefix="/smoke-test")` → mounts modular routes under `/smoke-test`  
- Root `/` endpoint returns a simple health message `{ "message": "OK" }`

---

## 3️⃣ CORS Middleware Explained

- **CORS** = Cross-Origin Resource Sharing  
- Browsers block requests from one origin to another unless allowed.  

Example problem:

Frontend: `http://localhost:3000`  
Backend: `http://localhost:8000`  

Without CORS → browser blocks requests.  
With CORS middleware → backend sends headers like:

```
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: *
Access-Control-Allow-Headers: *
```

- `"*"` allows all origins (good for dev, not production).  
- `allow_credentials=True` allows cookies and auth headers.  
- `allow_methods=["*"]` allows GET, POST, PUT, DELETE, etc.  
- `allow_headers=["*"]` allows custom headers like Authorization.

**Preflight requests:** Browser sends an OPTIONS request automatically; middleware handles it.

---

## 4️⃣ Debugging inside Docker

### Why
- Docker containers are isolated. Host VS Code cannot connect by default.  
- `debugpy` allows VS Code to attach to Python inside the container.

### How it works
1. Python app starts `debugpy.listen(("0.0.0.0", 5678))`  
2. Container forwards port 5678 to host via `devcontainer.json`  
3. VS Code attaches using port 5678  
4. You can set breakpoints, inspect variables, step through code

### Analogy
- Container = house  
- Python app = person inside  
- 0.0.0.0 = doors listening to everyone inside house  
- 5678 = specific door for VS Code  
- Port forwarding = guard letting VS Code reach that door  

---

## 5️⃣ 0.0.0.0 and 5678 in detail

- **`0.0.0.0`** → listen on all network interfaces (container-local + forwarded to host)  
- **`5678`** → TCP port used by debugpy for remote debugging  
- Together → allow VS Code on host to remotely debug the app inside Docker

---

## 6️⃣ Summary

- Dockerfile sets up **Python 3.12 dev environment** with essential tools  
- FastAPI app with **CORS enabled**, health check, and modular router  
- `debugpy` allows **remote debugging from VS Code**  
- `0.0.0.0:5678` → container listens for debugger connections on all interfaces and port 5678  
- `forwardPorts` in `devcontainer.json` maps container port to host for debugging

**Dev workflow:**  
1. Open project in VS Code  
2. Reopen in container → VS Code builds & attaches  
3. Start app → debugpy listens  
4. Attach debugger in VS Code → set breakpoints, step through code  

---

This setup allows **safe, productive development of FastAPI apps inside Docker** while supporting **modular code, hot reloading, and remote debugging**.
