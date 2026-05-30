# Dockerfile Explanation – LLM Service

This Dockerfile is used to build and run a Python-based LLM (Large Language Model) service. It uses a multi-stage build to create a lightweight and optimized Docker image by installing dependencies in one stage and running the application in another.

---

# Complete Dockerfile

```dockerfile
# Copyright The OpenTelemetry Authors
# SPDX-License-Identifier: Apache-2.0

FROM docker.io/library/python:3.12-alpine3.22 AS build-venv

RUN apk update && \
    apk add gcc g++ linux-headers

COPY ./src/llm/requirements.txt requirements.txt

RUN python -m venv venv && \
    venv/bin/pip install --no-cache-dir -r requirements.txt

FROM docker.io/library/python:3.12-alpine3.22

COPY --from=build-venv /venv/ /venv/

WORKDIR /app

COPY ./src/llm/app.py app.py
COPY ./src/llm/product-review-summaries/product-review-summaries.json product-review-summaries.json
COPY ./src/llm/product-review-summaries/inaccurate-product-review-summaries.json inaccurate-product-review-summaries.json

EXPOSE ${LLM_PORT}
ENTRYPOINT [ "/venv/bin/python", "app.py" ]
```

---

# Step-by-Step Explanation

## 1. Copyright Information

```dockerfile
# Copyright The OpenTelemetry Authors
# SPDX-License-Identifier: Apache-2.0
```

### Purpose

These are comment lines that:

* Identify the code owner.
* Specify the software license.
* Do not affect image creation.

---

## 2. First Stage: Build Environment

```dockerfile
FROM docker.io/library/python:3.12-alpine3.22 AS build-venv
```

### Purpose

Creates the first build stage named:

```text
build-venv
```

### Why?

This stage is used only for:

* Installing dependencies
* Creating a Python virtual environment

The final image will copy only the required files from this stage.

### Benefits

* Smaller image size
* Faster deployments
* Better security

---

## 3. Install Build Tools

```dockerfile
RUN apk update && \
    apk add gcc g++ linux-headers
```

### Purpose

Installs packages required to build Python dependencies.

### Package Details

| Package       | Purpose              |
| ------------- | -------------------- |
| gcc           | C compiler           |
| g++           | C++ compiler         |
| linux-headers | Linux kernel headers |

### Why Needed?

Some Python libraries contain native C/C++ code and must be compiled during installation.

Examples:

* numpy
* pandas
* cryptography
* grpc libraries

---

## 4. Copy Requirements File

```dockerfile
COPY ./src/llm/requirements.txt requirements.txt
```

### Purpose

Copies the dependency list into the container.

Example:

```text
requirements.txt
├── flask
├── requests
├── openai
└── grpcio
```

This file tells pip which packages to install.

---

## 5. Create Python Virtual Environment

```dockerfile
RUN python -m venv venv
```

### Purpose

Creates a Python virtual environment.

Generated directory:

```text
/venv
```

### Why Use Virtual Environments?

They:

* Isolate application dependencies
* Prevent package conflicts
* Keep the application self-contained

---

## 6. Install Python Dependencies

```dockerfile
venv/bin/pip install --no-cache-dir -r requirements.txt
```

### Purpose

Installs all packages listed in:

```text
requirements.txt
```

inside the virtual environment.

### Option Used

```text
--no-cache-dir
```

### Benefit

Prevents pip from storing installation cache files.

Result:

* Smaller image size
* Reduced storage usage

---

# Second Stage: Runtime Image

## 7. Create Final Runtime Image

```dockerfile
FROM docker.io/library/python:3.12-alpine3.22
```

### Purpose

Starts a new clean image.

### Why?

The build tools installed earlier are no longer required.

This keeps the final image lightweight.

---

## 8. Copy Virtual Environment

```dockerfile
COPY --from=build-venv /venv/ /venv/
```

### Purpose

Copies the completed Python virtual environment from the first stage.

### Result

The final image contains:

```text
/venv
├── Python interpreter
├── Installed libraries
└── Executable scripts
```

without including build dependencies.

---

## 9. Set Working Directory

```dockerfile
WORKDIR /app
```

### Purpose

Sets:

```text
/app
```

as the default directory.

All future commands execute from this location.

---

## 10. Copy Application Code

```dockerfile
COPY ./src/llm/app.py app.py
```

### Purpose

Copies the main Python application.

### Result

```text
/app/app.py
```

This file contains the application logic.

---

## 11. Copy Product Review Data

```dockerfile
COPY ./src/llm/product-review-summaries/product-review-summaries.json product-review-summaries.json
```

### Purpose

Copies review summary data into the container.

### Result

```text
/app/product-review-summaries.json
```

The application can read this file while running.

---

## 12. Copy Inaccurate Review Data

```dockerfile
COPY ./src/llm/product-review-summaries/inaccurate-product-review-summaries.json inaccurate-product-review-summaries.json
```

### Purpose

Copies an additional JSON file containing inaccurate review summaries.

### Result

```text
/app/inaccurate-product-review-summaries.json
```

This data may be used for:

* Testing
* Validation
* Demonstrations
* Model evaluation

---

## 13. Expose Application Port

```dockerfile
EXPOSE ${LLM_PORT}
```

### Purpose

Documents the port used by the application.

Example:

```bash
LLM_PORT=5000
```

Then the application listens on:

```text
Port 5000
```

### Note

`EXPOSE` does not publish the port.

Port mapping happens when starting the container:

```bash
docker run -p 5000:5000 llm-service
```

---

## 14. Application Startup Command

```dockerfile
ENTRYPOINT [ "/venv/bin/python", "app.py" ]
```

### Purpose

Defines the main process that starts when the container launches.

### Execution

Docker runs:

```bash
/venv/bin/python app.py
```

### Breakdown

| Component        | Purpose                                     |
| ---------------- | ------------------------------------------- |
| /venv/bin/python | Python interpreter from virtual environment |
| app.py           | Main application file                       |

### Why Use Python From /venv?

Ensures the application uses:

* Installed dependencies
* Correct package versions
* Isolated environment

---

# Multi-Stage Build Flow

```text
Stage 1 (build-venv)
│
├── Install gcc
├── Install g++
├── Install linux-headers
├── Create virtual environment
└── Install Python packages
         │
         ▼
Stage 2 (runtime)
│
├── Start clean Python image
├── Copy virtual environment
├── Copy application files
├── Copy JSON data files
└── Run app.py
```

---

# Summary

This Dockerfile:

1. Uses Python 3.12 on Alpine Linux.
2. Creates a separate build stage for dependency installation.
3. Installs required compilation tools.
4. Creates a Python virtual environment.
5. Installs application dependencies from requirements.txt.
6. Creates a clean runtime image.
7. Copies only the virtual environment and application files.
8. Adds JSON files used by the application.
9. Exposes the configured application port.
10. Starts the LLM application using the Python interpreter from the virtual environment.

This approach produces a smaller, cleaner, and more secure Docker image while ensuring all required Python dependencies are available.
