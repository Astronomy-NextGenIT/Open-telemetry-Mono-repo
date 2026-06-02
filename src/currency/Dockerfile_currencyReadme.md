# Dockerfile Explanation – Currency Service (C++)

## What is this Dockerfile?

This Dockerfile builds and runs the Currency Service of the OpenTelemetry Demo application.

Unlike other services:

* Accounting → .NET
* Ad → Java
* Checkout → Go
* Currency → C++

This service is written in **C++** and uses:

* CMake
* g++
* OpenTelemetry C++ SDK
* gRPC
* Protocol Buffers

The Dockerfile has **three stages**:

1. Base Stage
2. Builder Stage
3. Release Stage

---

# Overall Architecture

```text
Base Stage
     ↓
Builder Stage
     ↓
Build OpenTelemetry C++ SDK
     ↓
Build Currency Service
     ↓
Release Stage
     ↓
Run Currency Service
```

---

# Stage 1 : Base Stage

## Step 1: Use Alpine Linux

```dockerfile
FROM docker.io/library/alpine:3.21 AS base
```

### What is Alpine?

Alpine Linux is a lightweight Linux distribution.

Benefits:

* Small image size
* Fast downloads
* Lower attack surface
* Commonly used in containers

---

## Step 2: Install gRPC and Protobuf Libraries

```dockerfile
RUN apk update && \
    apk add grpc-dev protobuf-dev
```

### What is apk?

`apk` is Alpine's package manager.

Similar to:

```text
Ubuntu  → apt
RHEL    → yum
Alpine  → apk
```

---

### grpc-dev

Installs:

```text
gRPC headers
gRPC libraries
Development files
```

### Why?

Currency Service communicates with other microservices using gRPC.

---

### protobuf-dev

Installs:

```text
Protocol Buffer compiler libraries
Header files
```

### Why?

Needed to process `.proto` files.

---

# Stage 2 : Builder Stage

## Step 3: Start Builder Stage

```dockerfile
FROM base AS builder
```

### What it does

Uses the previously created base image.

Everything installed in Base Stage is available here.

---

## Step 4: Install Build Tools

```dockerfile
RUN apk add git cmake make g++ linux-headers
```

### What is installed?

| Package       | Purpose                   |
| ------------- | ------------------------- |
| git           | Clone repositories        |
| cmake         | Generate build files      |
| make          | Build projects            |
| g++           | C++ compiler              |
| linux-headers | Linux development headers |

---

### Why?

C++ code cannot be compiled without these tools.

---

## Step 5: Define OpenTelemetry Version

```dockerfile
ARG OPENTELEMETRY_CPP_VERSION
```

### What it does

Creates a variable.

Example:

```text
OPENTELEMETRY_CPP_VERSION=1.21.0
```

Used later to download the required SDK version.

---

# Building OpenTelemetry C++ SDK

This is the most important part.

---

## Step 6: Clone OpenTelemetry C++ Repository

```dockerfile
git clone --depth 1 --branch v${OPENTELEMETRY_CPP_VERSION} https://github.com/open-telemetry/opentelemetry-cpp
```

### What it does

Downloads OpenTelemetry C++ source code.

### Why --depth 1?

```text
Normal Clone → Entire Git History
Depth 1 Clone → Latest Version Only
```

Benefits:

* Faster download
* Smaller image

---

## Step 7: Create Build Directory

```dockerfile
mkdir build
cd build
```

### Why?

CMake best practice:

```text
Source Code
      ↓
Separate Build Folder
```

Keeps build files separate from source code.

---

## Step 8: Configure OpenTelemetry Build

```dockerfile
cmake ..
```

with options:

```dockerfile
-DCMAKE_CXX_STANDARD=17
```

Uses C++17 standard.

---

```dockerfile
-DCMAKE_BUILD_TYPE=Release
```

Optimized production build.

---

```dockerfile
-DBUILD_TESTING=OFF
```

Skips tests.

### Why?

Tests increase build time.

Not needed inside Docker image.

---

```dockerfile
-DWITH_EXAMPLES=OFF
```

Skips example programs.

### Why?

Examples are not needed in production.

---

```dockerfile
-DWITH_OTLP_GRPC=ON
```

Enables OTLP over gRPC.

### Why?

Allows traces and metrics to be sent to OpenTelemetry Collector.

---

```dockerfile
-DWITH_ABSEIL=ON
```

Enables Google's Abseil libraries.

Required by OpenTelemetry.

---

## Step 9: Build OpenTelemetry SDK

```dockerfile
make -j$(nproc || sysctl -n hw.ncpu || echo 1)
```

### What does -j do?

Builds using multiple CPU cores.

Example:

```text
1 CPU  → 1 job
8 CPUs → 8 jobs
```

Build becomes much faster.

---

## Step 10: Install OpenTelemetry SDK

```dockerfile
make install
```

### What it does

Installs SDK into:

```text
/usr/local
```

Result:

```text
/usr/local/lib
/usr/local/include
/usr/local/bin
```

These files are later used by Currency Service.

---

# Building Currency Service

## Step 11: Set Working Directory

```dockerfile
WORKDIR /currency
```

### What it does

Moves into Currency Service directory.

---

## Step 12: Copy Build Files

```dockerfile
COPY ./src/currency/build/ build/
```

Contains build-related files.

---

## Step 13: Copy Proto Files

```dockerfile
COPY ./src/currency/proto/ proto/
```

Contains Currency Service protobuf definitions.

---

## Step 14: Copy Source Code

```dockerfile
COPY ./src/currency/src/ src/
```

Contains actual C++ application code.

---

## Step 15: Copy CMake Files

```dockerfile
COPY ./src/currency/genproto/CMakeLists.txt genproto/CMakeLists.txt
COPY ./src/currency/CMakeLists.txt CMakeLists.txt
```

### What are CMakeLists.txt files?

Equivalent to:

```text
Maven → pom.xml
Gradle → build.gradle
NodeJS → package.json
C++ → CMakeLists.txt
```

Defines:

* Source files
* Dependencies
* Build instructions

---

## Step 16: Copy Shared Proto File

```dockerfile
COPY ./pb/demo.proto proto/demo.proto
```

### What is demo.proto?

Shared communication contract used across microservices.

---

## Step 17: Build Currency Service

```dockerfile
RUN mkdir -p build && cd build \
    && cmake .. \
    && make install
```

### What happens?

```text
Source Code
      ↓
CMake Generates Build Files
      ↓
Make Compiles Code
      ↓
Binary Created
      ↓
Installed to /usr/local/bin
```

Result:

```text
/usr/local/bin/currency
```

---

# Stage 3 : Release Stage

## Step 18: Create Runtime Image

```dockerfile
FROM base AS release
```

### Why?

Builder image contains:

```text
git
cmake
g++
make
```

These are not needed at runtime.

Using Base Stage keeps final image smaller.

---

## Step 19: Copy Installed Files

```dockerfile
COPY --from=builder /usr/local /usr/local
```

### What it does

Copies:

```text
OpenTelemetry SDK
Currency Binary
Libraries
Headers
```

from builder stage.

---

## Step 20: Expose Port

```dockerfile
EXPOSE ${CURRENCY_PORT}
```

### What it does

Makes Currency Service accessible on configured port.

Example:

```text
CURRENCY_PORT=7001
```

---

## Step 21: Start Currency Service

```dockerfile
ENTRYPOINT ["sh", "-c", "./usr/local/bin/currency ${CURRENCY_PORT}"]
```

### What it does

Starts the Currency Service executable.

Execution Flow:

```text
Container Starts
       ↓
Currency Binary Executes
       ↓
Port Passed as Argument
       ↓
Currency Service Starts
       ↓
OpenTelemetry Export Enabled
```

---

# Complete Build Flow

```text
Alpine Linux
      ↓
Install gRPC & Protobuf
      ↓
Install Build Tools
      ↓
Download OpenTelemetry C++ SDK
      ↓
Compile SDK
      ↓
Install SDK
      ↓
Copy Currency Source Code
      ↓
Compile Currency Service
      ↓
Create Runtime Image
      ↓
Copy Compiled Files
      ↓
Start Currency Service
```

# Key Learning Points

### Alpine Linux

Lightweight Linux distribution used for containers.

### gRPC

Framework used for microservice communication.

### Protocol Buffers

Define message formats shared between services.

### CMake

Build system generator for C++ projects.

### Make

Compiles the application.

### OpenTelemetry C++ SDK

Collects traces, metrics, and logs.

### Multi-Stage Build

Reduces image size by separating build and runtime environments.

### make install

Installs binaries and libraries into `/usr/local`.

# Summary

This Dockerfile builds both the OpenTelemetry C++ SDK and the Currency Service from source code. It then creates a lightweight runtime image containing only 
the required binaries and libraries.

Benefits:

* Production-ready build
* Smaller final image
* Uses OpenTelemetry C++ instrumentation
* Supports gRPC communication
* Uses Protocol Buffers
* Optimized release build
* Multi-stage architecture
