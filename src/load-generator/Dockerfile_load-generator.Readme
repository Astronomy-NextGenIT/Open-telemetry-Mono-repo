# Dockerfile Explanation – Load Generator Service

This Dockerfile is used to build and run a **Load Generator Service**. The service uses **Locust** for load testing and **Playwright with Chromium** for browser-based testing. It follows a multi-stage build approach to keep the final Docker image smaller and cleaner.

---

# Complete Dockerfile

```dockerfile
FROM python:3.12-slim-bookworm AS base

FROM base AS builder
RUN apt-get -qq update \
    && apt-get install -y --no-install-recommends g++ \
    && rm -rf /var/lib/apt/lists/*

COPY ./src/load-generator/requirements.txt .
RUN pip install --prefix="/reqs" -r requirements.txt

FROM base
WORKDIR /usr/src/app/
COPY --from=builder /reqs /usr/local
ENV PLAYWRIGHT_BROWSERS_PATH=/opt/pw-browsers
RUN playwright install --with-deps chromium
COPY ./src/load-generator/locustfile.py .
COPY ./src/load-generator/people.json .
ENTRYPOINT ["locust", "--skip-log-setup"]
```

---

# Step-by-Step Explanation

## 1. Create Base Image

```dockerfile
FROM python:3.12-slim-bookworm AS base
```

### Purpose

Uses Python 3.12 Slim Bookworm as the base image.

### Breakdown

| Part     | Meaning                    |
| -------- | -------------------------- |
| python   | Official Python image      |
| 3.12     | Python version             |
| slim     | Smaller image size         |
| bookworm | Debian 12 operating system |

### Why Use Slim?

Benefits:

* Smaller image size
* Faster downloads
* Less storage usage
* Improved security

---

## 2. Create Builder Stage

```dockerfile
FROM base AS builder
```

### Purpose

Creates a separate build stage called:

```text
builder
```

This stage is used only for:

* Installing dependencies
* Preparing Python packages

The final image copies only what is needed.

---

## 3. Install C++ Compiler

```dockerfile
RUN apt-get -qq update \
    && apt-get install -y --no-install-recommends g++ \
    && rm -rf /var/lib/apt/lists/*
```

### Purpose

Installs the GNU C++ compiler.

### Breakdown

#### Update Package List

```bash
apt-get -qq update
```

Downloads the latest package information.

Option:

```text
-qq
```

Means:

```text
Very quiet mode
```

Shows minimal output during build.

---

#### Install g++

```bash
apt-get install -y --no-install-recommends g++
```

Installs:

```text
g++
```

which is the GNU C++ compiler.

### Why Needed?

Some Python libraries contain C/C++ code and must be compiled during installation.

Examples:

* grpcio
* numpy
* cryptography

---

#### Clean Package Cache

```bash
rm -rf /var/lib/apt/lists/*
```

Deletes temporary package files.

### Benefit

Reduces Docker image size.

---

## 4. Copy Requirements File

```dockerfile
COPY ./src/load-generator/requirements.txt .
```

### Purpose

Copies the dependency list into the container.

Example:

```text
requirements.txt
├── locust
├── playwright
├── requests
└── grpcio
```

---

## 5. Install Python Dependencies

```dockerfile
RUN pip install --prefix="/reqs" -r requirements.txt
```

### Purpose

Installs all Python packages listed in:

```text
requirements.txt
```

### Installation Location

```text
/reqs
```

instead of the default Python directory.

### Why?

Allows the next stage to copy only installed packages.

Result:

```text
/reqs
├── bin
├── lib
└── site-packages
```

---

# Runtime Stage

## 6. Start Clean Runtime Image

```dockerfile
FROM base
```

### Purpose

Starts a new clean image based on:

```text
python:3.12-slim-bookworm
```

### Why?

Removes:

* Build tools
* Temporary files
* Compilation dependencies

This makes the final image smaller.

---

## 7. Set Working Directory

```dockerfile
WORKDIR /usr/src/app/
```

### Purpose

Sets:

```text
/usr/src/app/
```

as the default directory.

All future commands run from this location.

---

## 8. Copy Installed Packages

```dockerfile
COPY --from=builder /reqs /usr/local
```

### Purpose

Copies Python packages from the builder stage.

### Source

```text
/reqs
```

### Destination

```text
/usr/local
```

Result:

All required Python libraries become available in the final image.

---

## 9. Configure Playwright Browser Location

```dockerfile
ENV PLAYWRIGHT_BROWSERS_PATH=/opt/pw-browsers
```

### Purpose

Defines where Playwright stores browser binaries.

### Location

```text
/opt/pw-browsers
```

### Why?

Keeps browser files organized and easy to manage.

---

## 10. Install Chromium Browser

```dockerfile
RUN playwright install --with-deps chromium
```

### Purpose

Downloads and installs:

```text
Chromium Browser
```

along with required system dependencies.

### Why?

Playwright uses Chromium to simulate real user interactions.

Examples:

* Opening web pages
* Clicking buttons
* Filling forms
* Browser performance testing

---

## 11. Copy Load Test Script

```dockerfile
COPY ./src/load-generator/locustfile.py .
```

### Purpose

Copies the main Locust script.

### Result

```text
/usr/src/app/locustfile.py
```

### What Is locustfile.py?

Contains:

* User behavior definitions
* API calls
* Test scenarios
* Load testing logic

---

## 12. Copy Test Data File

```dockerfile
COPY ./src/load-generator/people.json .
```

### Purpose

Copies test data into the container.

### Result

```text
/usr/src/app/people.json
```

### Possible Usage

Contains sample users such as:

```json
[
  {
    "name": "John"
  },
  {
    "name": "Alice"
  }
]
```

Used during load testing.

---

## 13. Container Startup Command

```dockerfile
ENTRYPOINT ["locust", "--skip-log-setup"]
```

### Purpose

Defines the command executed when the container starts.

### Executed Command

```bash
locust --skip-log-setup
```

---

### What Is Locust?

Locust is a load-testing tool used to:

* Test application performance
* Simulate thousands of users
* Measure response times
* Detect bottlenecks

---

### Option Used

```text
--skip-log-setup
```

### Purpose

Prevents Locust from configuring its own logging.

### Why?

Allows logging to be managed externally through:

* Docker logs
* Kubernetes logs
* OpenTelemetry logging

---

# Multi-Stage Build Flow

```text
Stage 1 (builder)
│
├── Install g++
├── Copy requirements.txt
├── Install Python packages
└── Store packages in /reqs
         │
         ▼
Stage 2 (runtime)
│
├── Start clean Python image
├── Copy packages from builder
├── Install Chromium
├── Copy locustfile.py
├── Copy people.json
└── Start Locust
```

---

# Summary

This Dockerfile:

1. Uses Python 3.12 Slim Bookworm.
2. Uses a multi-stage build for optimization.
3. Installs required Python packages in a separate builder stage.
4. Copies only necessary dependencies to the final image.
5. Installs Playwright and Chromium browser.
6. Copies the Locust load-testing script.
7. Copies sample test data.
8. Configures Playwright browser storage.
9. Starts Locust automatically when the container launches.
10. Generates load against applications by simulating real user traffic and browser interactions.
