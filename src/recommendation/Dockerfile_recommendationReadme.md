# Dockerfile Explanation – Recommendation Service

This Dockerfile is used to build and run the **Recommendation Service**. The service is written in Python and uses **gRPC** for communication and **OpenTelemetry** for monitoring and observability.

The Dockerfile uses a **multi-stage build**, which helps create a smaller and cleaner production image.

---

# What Does This Service Do?

The Recommendation Service is responsible for:

* Receiving product recommendation requests
* Processing recommendation logic
* Returning recommended products
* Communicating through gRPC
* Sending traces and metrics using OpenTelemetry

---

# Complete Dockerfile

```dockerfile
FROM docker.io/library/python:3.14-alpine3.23 AS build-venv

RUN apk update && \
    apk add gcc g++ linux-headers

COPY ./src/recommendation/requirements.txt requirements.txt

RUN python -m venv venv && \
    venv/bin/pip install --no-cache-dir -r requirements.txt

RUN venv/bin/opentelemetry-bootstrap -a install

FROM docker.io/library/python:3.14-alpine3.23

COPY --from=build-venv /venv/ /venv/

WORKDIR /app

COPY ./src/recommendation/demo_pb2_grpc.py demo_pb2_grpc.py
COPY ./src/recommendation/demo_pb2.py demo_pb2.py
COPY ./src/recommendation/logger.py logger.py
COPY ./src/recommendation/metrics.py metrics.py
COPY ./src/recommendation/recommendation_server.py recommendation_server.py

EXPOSE ${RECOMMENDATION_PORT}

ENTRYPOINT [ "/venv/bin/opentelemetry-instrument", "/venv/bin/python", "recommendation_server.py" ]
```

---

# Stage 1 – Build Stage

The purpose of this stage is to:

* Create a Python virtual environment
* Install Python dependencies
* Install OpenTelemetry instrumentation

---

## Step 1: Use Python Base Image

```dockerfile
FROM docker.io/library/python:3.14-alpine3.23 AS build-venv
```

### Purpose

Uses Python 3.14 running on Alpine Linux.

### Why Alpine?

Benefits:

* Lightweight image
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

### Packages Installed

| Package       | Purpose              |
| ------------- | -------------------- |
| gcc           | C Compiler           |
| g++           | C++ Compiler         |
| linux-headers | Linux system headers |

### Why Needed?

Some Python libraries require compilation during installation.

Examples:

* grpcio
* protobuf
* cryptography

---

## Step 3: Copy Requirements File

```dockerfile
COPY ./src/recommendation/requirements.txt requirements.txt
```

### Purpose

Copies the dependency list into the container.

The file contains all Python packages needed by the Recommendation Service.

---

## Step 4: Create Virtual Environment

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

Benefits:

* Keeps dependencies isolated
* Avoids conflicts
* Makes deployment easier

---

## Step 5: Install Dependencies

```dockerfile
venv/bin/pip install --no-cache-dir -r requirements.txt
```

### Purpose

Installs all Python packages listed in:

```text
requirements.txt
```

### Option Used

```text
--no-cache-dir
```

### Benefit

Prevents temporary package files from being stored.

Result:

* Smaller image size
* Less disk usage

---

## Step 6: Install OpenTelemetry Instrumentation

```dockerfile
RUN venv/bin/opentelemetry-bootstrap -a install
```

### Purpose

Automatically installs OpenTelemetry instrumentation packages.

### What Does It Do?

OpenTelemetry scans installed libraries and adds telemetry support.

Examples:

```text
gRPC
Requests
Flask
Django
Database libraries
```

### Benefit

The application can generate traces and metrics automatically without adding extra monitoring code.

---

# Stage 2 – Runtime Stage

This stage creates the final image used in production.

---

## Step 7: Create Runtime Image

```dockerfile
FROM docker.io/library/python:3.14-alpine3.23
```

### Purpose

Starts a fresh Python image.

### Why?

The build tools are no longer needed.

This keeps the final image cleaner and smaller.

---

## Step 8: Copy Virtual Environment

```dockerfile
COPY --from=build-venv /venv/ /venv/
```

### Purpose

Copies the virtual environment created in Stage 1.

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

as the working directory.

All application files will be stored here.

---

## Step 10: Copy gRPC Generated Files

### Copy Service Definitions

```dockerfile
COPY ./src/recommendation/demo_pb2_grpc.py demo_pb2_grpc.py
```

### Purpose

Contains generated gRPC service definitions.

Used for communication between services.

---

### Copy Protocol Buffer Classes

```dockerfile
COPY ./src/recommendation/demo_pb2.py demo_pb2.py
```

### Purpose

Contains generated Protocol Buffer message classes.

Examples:

```text
RecommendationRequest
RecommendationResponse
```

---

## Step 11: Copy Logger Module

```dockerfile
COPY ./src/recommendation/logger.py logger.py
```

### Purpose

Handles application logging.

Examples:

* Information logs
* Warning logs
* Error logs

---

## Step 12: Copy Metrics Module

```dockerfile
COPY ./src/recommendation/metrics.py metrics.py
```

### Purpose

Collects custom metrics.

Examples:

* Request count
* Response count
* Processing time
* Error count

---

## Step 13: Copy Main Application

```dockerfile
COPY ./src/recommendation/recommendation_server.py recommendation_server.py
```

### Purpose

This is the main Recommendation Service application.

Responsible for:

* Starting the gRPC server
* Processing recommendation requests
* Returning recommended products

---

## Step 14: Expose Application Port

```dockerfile
EXPOSE ${RECOMMENDATION_PORT}
```

### Purpose

Documents which port the service uses.

Example:

```bash
RECOMMENDATION_PORT=8080
```

Application listens on:

```text
Port 8080
```

### Important

EXPOSE does not publish the port.

Port mapping is done when running the container.

Example:

```bash
docker run -p 8080:8080 recommendation-service
```

---

## Step 15: Start Application with OpenTelemetry

```dockerfile
ENTRYPOINT [ "/venv/bin/opentelemetry-instrument", "/venv/bin/python", "recommendation_server.py" ]
```

### Purpose

Starts the Recommendation Service with automatic OpenTelemetry monitoring.

### What Happens?

#### Step 1

OpenTelemetry starts first:

```text
opentelemetry-instrument
```

#### Step 2

Instrumentation is automatically applied.

#### Step 3

Python starts:

```text
recommendation_server.py
```

#### Step 4

The Recommendation Service begins accepting requests.

#### Step 5

Traces and metrics are automatically collected and exported.

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
Start Recommendation Service
       │
       ▼
Receive gRPC Requests
       │
       ▼
Generate Recommendations
       │
       ▼
Create Metrics & Traces
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
5. Installs OpenTelemetry instrumentation.

## Stage 2 – Runtime Stage

6. Creates a clean Python runtime image.
7. Copies the virtual environment.
8. Copies gRPC generated files.
9. Copies logging and metrics modules.
10. Copies the main Recommendation Service application.
11. Exposes the Recommendation Service port.
12. Starts the application using OpenTelemetry instrumentation.

### Final Result

The container runs a Python-based Recommendation Service that:

* Receives recommendation requests
* Returns product recommendations
* Communicates using gRPC
* Automatically generates traces and metrics
* Sends telemetry data to OpenTelemetry for monitoring and observability
