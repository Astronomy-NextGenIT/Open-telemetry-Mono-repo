# Dockerfile Explanation – Shipping Service

This Dockerfile is used to build and run the **Shipping Service**, which is written in **Rust**.

The Dockerfile supports both:

* AMD64 (x86_64) systems
* ARM64 systems

It uses a **multi-stage build** to create a small and secure production image.

---

# What Does This Service Do?

The Shipping Service is responsible for:

* Processing shipping requests
* Calculating shipping information
* Handling order shipment operations
* Communicating with other microservices

The application is written in Rust and compiled into a standalone executable.

---

# Complete Dockerfile

```dockerfile
FROM --platform=${BUILDPLATFORM} docker.io/library/rust:1.91 AS builder

ARG TARGETARCH
ARG TARGETPLATFORM
ARG BUILDPLATFORM

RUN echo Building on ${BUILDPLATFORM} for ${TARGETPLATFORM}

RUN apt-get update && \
    if [ "${TARGETPLATFORM}" = "linux/arm64" ] && [ "${BUILDPLATFORM}" != "linux/arm64" ] ; then \
        apt-get install --no-install-recommends -y g++-aarch64-linux-gnu libc6-dev-arm64-cross libprotobuf-dev protobuf-compiler ca-certificates && \
        rustup target add aarch64-unknown-linux-gnu; \
    else \
        apt-get install --no-install-recommends -y g++ libc6-dev libprotobuf-dev protobuf-compiler ca-certificates; \
    fi

WORKDIR /app/

COPY /src/shipping/ /app/

RUN if [ "${TARGETPLATFORM}" = "linux/arm64" ] && [ "${BUILDPLATFORM}" != "linux/arm64" ] ; then \
        env CARGO_TARGET_AARCH64_UNKNOWN_LINUX_GNU_LINKER=aarch64-linux-gnu-gcc \
            CC_aarch64_unknown_linux_gnu=aarch64-linux-gnu-gcc \
            CXX_aarch64_unknown_linux_gnu=aarch64-linux-gnu-g++ \
        cargo build -r --target aarch64-unknown-linux-gnu && \
        cp /app/target/aarch64-unknown-linux-gnu/release/shipping /app/target/release/shipping; \
    else \
        cargo build -r; \
    fi

FROM gcr.io/distroless/cc-debian13:nonroot

WORKDIR /app

COPY --from=builder /app/target/release/shipping shipping

EXPOSE ${SHIPPING_PORT}

CMD ["./shipping"]
```

---

# Understanding Multi-Stage Build

This Dockerfile has two stages:

## Stage 1 – Builder Stage

Purpose:

```text
Compile Rust application
```

Contains:

* Rust compiler
* Cargo
* Build tools
* Protobuf compiler

---

## Stage 2 – Runtime Stage

Purpose:

```text
Run Shipping Service
```

Contains only:

* Shipping executable

This makes the final image:

* Smaller
* Faster
* More secure

---

# Stage 1 – Builder Stage

---

## Step 1: Use Rust Base Image

```dockerfile
FROM --platform=${BUILDPLATFORM} docker.io/library/rust:1.91 AS builder
```

### Purpose

Uses the official Rust image for building the application.

### Breakdown

| Part    | Meaning             |
| ------- | ------------------- |
| rust    | Official Rust image |
| 1.91    | Rust version        |
| builder | Build stage name    |

---

### What Does --platform=${BUILDPLATFORM} Mean?

Docker automatically sets:

```text
BUILDPLATFORM
```

Example:

```text
linux/amd64
```

This tells Docker:

```text
Use the architecture of the machine doing the build.
```

---

## Step 2: Define Build Variables

```dockerfile
ARG TARGETARCH
ARG TARGETPLATFORM
ARG BUILDPLATFORM
```

### Purpose

These variables are provided by Docker Buildx.

### Example

```text
BUILDPLATFORM=linux/amd64
TARGETPLATFORM=linux/arm64
TARGETARCH=arm64
```

---

### Why Needed?

They allow the same Dockerfile to build images for different CPU architectures.

Examples:

* Intel CPUs
* AMD CPUs
* ARM processors
* AWS Graviton

---

## Step 3: Display Build Information

```dockerfile
RUN echo Building on ${BUILDPLATFORM} for ${TARGETPLATFORM}
```

### Example Output

```text
Building on linux/amd64 for linux/arm64
```

### Purpose

Helps developers understand:

* Which platform is building
* Which platform is being targeted

Useful for troubleshooting.

---

## Step 4: Install Build Dependencies

```dockerfile
RUN apt-get update && ...
```

### Purpose

Installs all tools required to build the Rust application.

---

### Scenario 1: Cross Compilation

If:

```text
Building on AMD64
Targeting ARM64
```

Example:

```text
Developer Laptop (AMD64)
        ↓
Build ARM64 Image
```

Docker installs:

```text
g++-aarch64-linux-gnu
libc6-dev-arm64-cross
libprotobuf-dev
protobuf-compiler
ca-certificates
```

---

### What Are These Packages?

#### g++-aarch64-linux-gnu

ARM64 C/C++ compiler.

Used to build ARM binaries on non-ARM systems.

---

#### libc6-dev-arm64-cross

ARM64 C libraries.

Required when compiling ARM applications.

---

#### libprotobuf-dev

Protocol Buffers development libraries.

Used by gRPC services.

---

#### protobuf-compiler

Provides:

```text
protoc
```

which generates source code from:

```text
.proto
```

files.

---

#### ca-certificates

Trusted SSL certificates.

Used when downloading packages securely.

---

### Add ARM Rust Target

```dockerfile
rustup target add aarch64-unknown-linux-gnu
```

### Purpose

Adds ARM64 support to Rust compiler.

Without this step:

```text
Rust cannot build ARM binaries.
```

---

### Scenario 2: Native Build

If:

```text
BUILDPLATFORM = TARGETPLATFORM
```

Docker installs standard build packages only.

No cross-compilation tools are required.

---

## Step 5: Set Working Directory

```dockerfile
WORKDIR /app/
```

### Purpose

Sets:

```text
/app/
```

as the working directory.

---

## Step 6: Copy Source Code

```dockerfile
COPY /src/shipping/ /app/
```

### Purpose

Copies the Shipping Service source code.

Possible files:

```text
Cargo.toml
Cargo.lock
src/
proto/
build.rs
```

---

## Step 7: Build Rust Application

```dockerfile
RUN if [ "${TARGETPLATFORM}" = "linux/arm64" ] ...
```

### Purpose

Compiles the Shipping Service.

---

### ARM64 Build

Docker configures:

```text
aarch64-linux-gnu-gcc
aarch64-linux-gnu-g++
```

as compilers.

Then executes:

```bash
cargo build -r --target aarch64-unknown-linux-gnu
```

---

### What Does -r Mean?

```text
Release Mode
```

Equivalent to:

```text
Optimized Production Build
```

Benefits:

* Faster execution
* Smaller binary
* Better performance

---

### Copy ARM Binary

After compilation:

```dockerfile
cp /app/target/aarch64-unknown-linux-gnu/release/shipping \
   /app/target/release/shipping
```

### Why?

Creates a consistent file location.

Later stages always copy:

```text
/app/target/release/shipping
```

regardless of architecture.

---

### Native Build

For AMD64:

```bash
cargo build -r
```

builds the binary directly.

Output:

```text
/app/target/release/shipping
```

---

# Stage 2 – Runtime Stage

---

## Step 8: Use Distroless Runtime Image

```dockerfile
FROM gcr.io/distroless/cc-debian13:nonroot
```

### Purpose

Creates the final production image.

---

### What Is Distroless?

A Distroless image contains only:

* Required runtime libraries
* Application binary

It does NOT contain:

* Shell
* Package manager
* Debugging tools

---

### Benefits

* Smaller image size
* Faster startup
* Improved security
* Fewer vulnerabilities

---

### What Does nonroot Mean?

Application runs as a non-root user.

Benefits:

* Better security
* Reduced permissions
* Safer production deployment

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

as the runtime directory.

---

## Step 10: Copy Shipping Binary

```dockerfile
COPY --from=builder /app/target/release/shipping shipping
```

### Purpose

Copies the compiled executable from Stage 1.

Result:

```text
/app/shipping
```

becomes available in the final image.

---

## Step 11: Expose Service Port

```dockerfile
EXPOSE ${SHIPPING_PORT}
```

### Purpose

Documents which port the Shipping Service uses.

Example:

```bash
SHIPPING_PORT=50051
```

Application listens on:

```text
Port 50051
```

---

## Step 12: Start the Application

```dockerfile
CMD ["./shipping"]
```

### Purpose

Starts the Shipping Service when the container launches.

Executed command:

```bash
./shipping
```

---

# Build Flow

```text
Rust Source Code
        │
        ▼
Install Dependencies
        │
        ▼
Compile Rust Application
        │
        ▼
Create Shipping Binary
        │
        ▼
Copy Binary To Runtime Image
        │
        ▼
Start Shipping Service
```

---

# Summary

## Builder Stage

1. Uses Rust 1.91 image.
2. Supports AMD64 and ARM64 builds.
3. Installs Rust and protobuf build dependencies.
4. Configures cross-compilation when needed.
5. Compiles the Shipping Service binary.

## Runtime Stage

6. Uses Distroless Debian image.
7. Runs as a non-root user.
8. Copies only the compiled executable.
9. Exposes the Shipping Service port.
10. Starts the Shipping Service.

### Final Result

The container runs a lightweight Rust-based Shipping Service that:

* Supports multiple CPU architectures
* Uses gRPC and Protocol Buffers
* Runs securely as a non-root user
* Uses a small production-ready image
* Starts using the compiled `shipping` executable
