# Dockerfile Explanation – Product Reviews Service

This Dockerfile is used to build and run the **Product Reviews Service**. The service is written in Python and uses **OpenTelemetry** for automatic monitoring and observability.

The Dockerfile follows a **multi-stage build approach**, which helps create a smaller and cleaner production image.

---

# What Does This Service Do?

The Product Reviews Service is responsible for:

* Receiving product review requests
* Returning product review data
* Communicating through gRPC
* Sending telemetry data (traces, metrics, logs) using OpenTelemetry

---

# Complete Dockerfile

```dockerfile
FROM docker.io/library/python:3.14-alpine3.23 AS build-venv

RUN apk update && \
    apk add gcc g++ linux-headers

COPY ./src/product-reviews/requirements.txt requirements.txt

RUN python -m venv venv && \
    venv/bin/pip install --no-cache-dir -r requirements.txt

RUN venv/bin/opentelemetry-bootstrap -a install

FROM docker.io/library/python:3.14-alpine3.23

COPY --from=build-venv /venv/ /venv/

WORKDIR /app

COPY ./src/product-reviews/demo_pb2_grpc.py demo_pb2_grpc.py
COPY ./src/product-reviews/demo_pb2.py demo_pb2.py
COPY ./src/product-reviews/product_reviews_server.py product_reviews_server.py
COPY ./src/product-reviews/database.py database.py
COPY ./src/product-reviews/metrics.py metrics.py

EXPOSE ${PRODUCT_REVIEWS_PORT}

ENTRYPOINT [ "/venv/bin/opentelemetry-instrument", "/venv/bin/python", "product_reviews_server.py" ]
```

---

# Stage 1 – Build Stage

The purpose of this stage is to:

* Install Python packages
* Install OpenTelemetry dependencies
* Create a virtual environment

---

## Step 1: Use Python Base Image

```dockerfile
FROM docker.io/library/python:3.14-alpine3.23 AS build-venv
```

### Purpose

Uses Python 3.14 running on Alpine Linux.

### Breakdown

| Part       | Meaning                        |
| ---------- | ------------------------------ |
| python     | Official Python image          |
| 3.14       | Python version                 |
| alpine3.23 | Lightweight Linux distribution |
| build-venv | Name of build stage            |

### Why Alpine?

Benefits:

* Small image size
* Faster downloads
* Lower storage usage

---

## Step 2: Install Build Tools

```dockerfile
RUN apk update && \
    apk add gcc g++ linux-headers
```

### Purpose

Installs tools required to compile Python packages.

### Package Explanation

| Package       | Purpose              |
| ------------- | -------------------- |
| gcc           | C Compiler           |
| g++           | C++ Compiler         |
| linux-headers | Linux system headers |

### Why Needed?

Some Python libraries contain C/C++ code and must be compiled during installation.

Examples:

* grpcio
* protobuf
* cryptography

---

## Step 3: Copy Requirements File

```dockerfile
COPY ./src/product-reviews/requirements.txt requirements.txt
```

### Purpose

Copies the dependency list into the container.

Example:

```text
requirements.txt
├── grpcio
├── opentelemetry-sdk
├── protobuf
└── requests
```

---

## Step 4: Create Python Virtual Environment

```dockerfile
RUN python -m venv venv
```

### Purpose

Creates an isolated Python environment.

Result:

```text
/venv
```

### Why Use Virtual Environment?

It keeps:

* Python packages isolated
* Dependencies organized
* Application portable

---

## Step 5: Install Python Dependencies

```dockerfile
venv/bin/pip install --no-cache-dir -r requirements.txt
```

### Purpose

Installs all packages listed in:

```text
requirements.txt
```

### Option Used

```text
--no-cache-dir
```

### Benefit

Does not store temporary installation files.

Result:

* Smaller Docker image
* Less disk usage

---

## Step 6: Install OpenTelemetry Instrumentation

```dockerfile
RUN venv/bin/opentelemetry-bootstrap -a install
```

### Purpose

Automatically installs OpenTelemetry instrumentation packages.

### What is OpenTelemetry Bootstrap?

It scans installed libraries and installs telemetry support for them.

### Example

If application uses:

```text
gRPC
Requests
Flask
Django
```

OpenTelemetry installs instrumentation packages automatically.

### Benefit

No code changes required for telemetry collection.

---

# Stage 2 – Runtime Stage

This stage creates the final lightweight production image.

---

## Step 7: Create Runtime Image

```dockerfile
FROM docker.io/library/python:3.14-alpine3.23
```

### Purpose

Starts a fresh Python image.

### Why?

The build tools are no longer needed.

This keeps the final image smaller.

---

## Step 8: Copy Virtual Environment

```dockerfile
COPY --from=build-venv /venv/ /venv/
```

### Purpose

Copies the completed virtual environment from Stage 1.

Result:

```text
/venv
├── Python packages
├── OpenTelemetry packages
└── Python executables
```

---

## Step 9: Set Working Directory

```dockerfile
WORKDIR /app
```

### Purpose

Sets:

```text
/app
```

as the application's working directory.

All application files will be stored here.

---

## Step 10: Copy gRPC Generated Files

### Copy gRPC Service Code

```dockerfile
COPY ./src/product-reviews/demo_pb2_grpc.py demo_pb2_grpc.py
```

### Purpose

Contains generated gRPC service definitions.

Used for:

* Client communication
* Server communication

---

### Copy Protocol Buffer Classes

```dockerfile
COPY ./src/product-reviews/demo_pb2.py demo_pb2.py
```

### Purpose

Contains generated Protocol Buffer message classes.

Examples:

```text
ReviewRequest
ReviewResponse
```

---

## Step 11: Copy Main Application

```dockerfile
COPY ./src/product-reviews/product_reviews_server.py product_reviews_server.py
```

### Purpose

Main server application.

Responsible for:

* Starting gRPC server
* Processing requests
* Returning product reviews

This is the most important file in the service.

---

## Step 12: Copy Database Module

```dockerfile
COPY ./src/product-reviews/database.py database.py
```

### Purpose

Handles data operations.

Examples:

* Reading review data
* Storing review information
* Database communication

---

## Step 13: Copy Metrics Module

```dockerfile
COPY ./src/product-reviews/metrics.py metrics.py
```

### Purpose

Collects custom application metrics.

Examples:

* Request count
* Response count
* Error count
* Processing time

---

## Step 14: Expose Application Port

```dockerfile
EXPOSE ${PRODUCT_REVIEWS_PORT}
```

### Purpose

Documents the port used by the service.

Example:

```bash
PRODUCT_REVIEWS_PORT=8080
```

Application listens on:

```text
Port 8080
```

### Important

EXPOSE does not publish the port.

Publishing happens when running the container:

```bash
docker run -p 8080:8080 product-reviews
```

---

## Step 15: Start Application with OpenTelemetry

```dockerfile
ENTRYPOINT [ "/venv/bin/opentelemetry-instrument", "/venv/bin/python", "product_reviews_server.py" ]
```

### Purpose

Starts the Product Reviews Service with automatic OpenTelemetry monitoring.

---

### What Happens?

#### Step 1

OpenTelemetry starts first:

```text
opentelemetry-instrument
```

#### Step 2

It automatically instruments the application.

#### Step 3

Python starts:

```text
product_reviews_server.py
```

#### Step 4

Telemetry data begins flowing to the OpenTelemetry Collector.

---

# Startup Flow

```text
Container Starts
       │
       ▼
Load OpenTelemetry Instrumentation
       │
       ▼
Start Python Runtime
       │
       ▼
Start Product Reviews Server
       │
       ▼
Receive gRPC Requests
       │
       ▼
Generate Metrics & Traces
       │
       ▼
Send Data to OpenTelemetry Collector
```

---

# Summary

## Stage 1 – Build Stage

1. Uses Python 3.14 Alpine image.
2. Installs build tools.
3. Creates a virtual environment.
4. Installs Python dependencies.
5. Installs OpenTelemetry instrumentation automatically.

## Stage 2 – Runtime Stage

6. Creates a clean Python runtime image.
7. Copies the virtual environment.
8. Copies gRPC generated files.
9. Copies the main server code.
10. Copies database and metrics modules.
11. Exposes the Product Reviews service port.
12. Starts the application using OpenTelemetry instrumentation.

### Final Result

The container runs a Python gRPC Product Reviews Service that automatically generates and exports:

* Traces
* Metrics
* Logs

without requiring additional monitoring code in the application.
