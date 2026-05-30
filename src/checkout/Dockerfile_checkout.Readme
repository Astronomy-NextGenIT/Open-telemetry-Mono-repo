# Dockerfile Explanation – Checkout Service (Go)

## What is this Dockerfile?

This Dockerfile is used to build and run the Checkout Service of the OpenTelemetry Demo application.

The Checkout Service is written in **Go (Golang)** and uses a **Multi-Stage Build** approach.

The main purpose of this Dockerfile is to:

1. Download Go dependencies.
2. Compile the Go application.
3. Create a very small runtime image.
4. Run the Checkout service securely.

---

# Understanding the Multi-Stage Build

This Dockerfile contains **two stages**:

### Stage 1 – Builder Stage

Used for:

* Downloading dependencies
* Compiling source code
* Creating executable binary

### Stage 2 – Runtime Stage

Used for:

* Running the compiled application

### Why use Multi-Stage Builds?

Without Multi-Stage Builds:

```text
Docker Image
 ├── Source Code
 ├── Go Compiler
 ├── Build Tools
 ├── Cache Files
 └── Application
```

Image becomes large.

With Multi-Stage Builds:

```text
Docker Image
 └── Application Binary Only
```

Benefits:

* Smaller image size
* Faster deployment
* Better security
* Lower storage usage

---

# Builder Stage

## Step 1: Use Go Builder Image

```dockerfile
FROM golang:1.25-bookworm AS builder
```

### What it does

Downloads the Go 1.25 image based on Debian Bookworm.

### What is included?

* Go compiler
* Go modules support
* Build tools
* Standard libraries

### Why it is needed?

The Checkout application must be compiled before it can run.

---

## Step 2: Set Working Directory

```dockerfile
WORKDIR /usr/src/app/
```

### What it does

Creates and moves into:

```text
/usr/src/app
```

inside the container.

All following commands execute from this directory.

---

## Step 3: Copy Dependency Files

```dockerfile
COPY ./src/checkout/go.mod go.mod
COPY ./src/checkout/go.sum go.sum
```

### What are these files?

#### go.mod

Contains:

* Project name
* Required packages
* Dependency versions

Example:

```text
github.com/grpc/grpc-go
github.com/open-telemetry/opentelemetry-go
```

#### go.sum

Contains:

* Checksums of downloaded packages

Used for dependency verification and security.

---

## Step 4: Download Dependencies

```dockerfile
RUN go mod download
```

### What it does

Downloads all Go packages defined in:

```text
go.mod
```

### Why is it done before copying source code?

Docker caches layers.

If source code changes but dependencies remain the same:

```text
Dependencies
    ↓
Already Cached
    ↓
Faster Build
```

This significantly improves build performance.

---

## Step 5: Copy Generated Proto Files

```dockerfile
COPY ./src/checkout/genproto/oteldemo/ genproto/oteldemo/
```

### What it does

Copies generated protobuf code.

### Why is it needed?

These files are generated from `.proto` definitions and are used for:

* gRPC communication
* Service-to-service messaging

---

## Step 6: Copy Kafka Package

```dockerfile
COPY ./src/checkout/kafka/ kafka/
```

### What it does

Copies Kafka-related code.

### Purpose

Used for:

* Producing messages
* Consuming messages
* Event-driven communication

Example:

```text
Order Created
      ↓
Kafka Topic
      ↓
Other Services Receive Event
```

---

## Step 7: Copy Money Package

```dockerfile
COPY ./src/checkout/money/ money/
```

### What it does

Copies money and currency-related logic.

### Typical responsibilities

* Price calculations
* Currency conversion
* Order totals

---

## Step 8: Copy Main Application File

```dockerfile
COPY ./src/checkout/main.go main.go
```

### What is main.go?

The starting point of the Go application.

Similar to:

```text
Java   → main()
C#     → Program.cs
Go     → main.go
```

When the service starts, execution begins here.

---

## Step 9: Build the Application

```dockerfile
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags "-s -w" -o checkout main.go
```

This is the most important step.

---

### CGO_ENABLED=0

```dockerfile
CGO_ENABLED=0
```

Disables C language dependencies.

Benefits:

* Smaller binary
* Easier deployment
* Better portability

---

### GOOS=linux

```dockerfile
GOOS=linux
```

Builds the application specifically for Linux.

---

### go build

```dockerfile
go build
```

Compiles source code into an executable.

---

### -ldflags "-s -w"

```dockerfile
-ldflags "-s -w"
```

Removes:

* Debug information
* Symbol tables

Benefits:

```text
Smaller Binary
      ↓
Smaller Docker Image
      ↓
Faster Deployment
```

---

### -o checkout

```dockerfile
-o checkout
```

Creates executable:

```text
checkout
```

Result:

```text
/usr/src/app/checkout
```

---

# Runtime Stage

The build is complete.

Now Docker creates a minimal runtime image.

---

## Step 10: Use Distroless Runtime Image

```dockerfile
FROM gcr.io/distroless/static-debian12:nonroot
```

### What is Distroless?

Distroless images contain:

* Application
* Required runtime libraries

Only.

They do NOT contain:

* Shell
* Package manager
* Compiler
* Extra utilities

---

### Why Distroless?

Benefits:

#### Smaller Image

Much smaller than Ubuntu or Debian.

#### Better Security

Fewer packages = fewer vulnerabilities.

#### Production Focused

Designed specifically for running applications.

---

### What does "nonroot" mean?

```dockerfile
:nonroot
```

The application runs as a non-root user.

Benefits:

* Better security
* Reduced attack surface
* Kubernetes best practice

---

## Step 11: Set Runtime Working Directory

```dockerfile
WORKDIR /usr/src/app/
```

### What it does

Creates application directory inside runtime container.

---

## Step 12: Copy Executable

```dockerfile
COPY --from=builder /usr/src/app/checkout/ ./
```

### What it does

Copies the compiled binary from builder stage.

Result:

```text
/usr/src/app
 └── checkout
```

Notice:

* No source code
* No Go compiler
* No build tools

Only the executable.

---

## Step 13: Expose Application Port

```dockerfile
EXPOSE ${CHECKOUT_PORT}
```

### What it does

Documents which port Checkout Service listens on.

Example:

```text
CHECKOUT_PORT=5050
```

Other services can connect through this port.

---

## Step 14: Start the Application

```dockerfile
ENTRYPOINT [ "./checkout" ]
```

### What it does

Starts the Checkout Service.

Execution flow:

```text
Container Starts
        ↓
checkout Binary Executes
        ↓
Checkout Service Starts
        ↓
Receives Requests
```

---

# Overall Dockerfile Flow

```text
Copy go.mod and go.sum
          ↓
Download Dependencies
          ↓
Copy Source Code
          ↓
Build Go Binary
          ↓
Create Distroless Runtime Image
          ↓
Copy Binary
          ↓
Expose Port
          ↓
Start Checkout Service
```

# Key Concepts Used

## go.mod

Defines project dependencies.

## go.sum

Stores dependency checksums.

## go mod download

Downloads all required packages.

## go build

Compiles Go code into executable.

## CGO_ENABLED=0

Creates a portable static binary.

## Distroless Image

Ultra-small image containing only runtime requirements.

## Non-Root User

Improves container security.

## ENTRYPOINT

Command executed when container starts.

# Summary

This Dockerfile builds the Checkout Service using Go and packages it into a highly optimized production image.

Major advantages:

* Multi-stage build
* Very small image size
* Fast startup
* High security
* Distroless runtime
* Runs as non-root user
* Production-ready for Kubernetes and Docker environments

Compared to the .NET and Java services, this is one of the most lightweight Dockerfiles because the final image contains only a single compiled Go binary and almost nothing else.
