# Dockerfile Explanation – Product Catalog Service

This Dockerfile is used to build and run the **Product Catalog Service**, which is written in Go (Golang). It follows a **multi-stage build** approach to create a small, secure, and production-ready Docker image.

---

# Complete Dockerfile

```dockerfile
FROM golang:1.25-bookworm AS builder

WORKDIR /usr/src/app/

COPY ./src/product-catalog/go.mod go.mod
COPY ./src/product-catalog/go.sum go.sum

RUN go mod download

COPY ./src/product-catalog/genproto/oteldemo/ genproto/oteldemo/
COPY ./src/product-catalog/main.go main.go

RUN CGO_ENABLED=0 GOOS=linux GO111MODULE=on go build -ldflags "-s -w" -o product-catalog main.go

FROM gcr.io/distroless/static-debian12:nonroot

WORKDIR /usr/src/app/

COPY --from=builder /usr/src/app/product-catalog/ ./

EXPOSE ${PRODUCT_CATALOG_PORT}
ENTRYPOINT [ "./product-catalog" ]
```

---

# Stage 1 – Build Stage

The first stage is responsible for downloading dependencies and compiling the Go application.

---

## Step 1: Use Go Base Image

```dockerfile
FROM golang:1.25-bookworm AS builder
```

### Purpose

Uses the official Go image as the build environment.

### Breakdown

| Part     | Meaning             |
| -------- | ------------------- |
| golang   | Official Go image   |
| 1.25     | Go version          |
| bookworm | Debian 12 OS        |
| builder  | Name of build stage |

### Why?

This image contains:

* Go compiler
* Go tools
* Build utilities

needed to compile the application.

---

## Step 2: Set Working Directory

```dockerfile
WORKDIR /usr/src/app/
```

### Purpose

Creates and sets the working directory.

All following commands execute from:

```text
/usr/src/app/
```

---

## Step 3: Copy Dependency Files

```dockerfile
COPY ./src/product-catalog/go.mod go.mod
COPY ./src/product-catalog/go.sum go.sum
```

### Purpose

Copies Go dependency files.

### go.mod

Contains:

* Module name
* Required packages
* Dependency versions

### go.sum

Contains:

* Checksums of dependencies
* Dependency verification information

---

## Step 4: Download Dependencies

```dockerfile
RUN go mod download
```

### Purpose

Downloads all Go packages defined in:

```text
go.mod
```

### Benefit

Dependencies are downloaded before source code is copied.

This improves Docker layer caching and speeds up future builds.

---

## Step 5: Copy Application Source Code

```dockerfile
COPY ./src/product-catalog/genproto/oteldemo/ genproto/oteldemo/
COPY ./src/product-catalog/main.go main.go
```

### Purpose

Copies the application source code.

### Files Copied

#### Generated Protocol Files

```text
genproto/oteldemo/
```

Contains generated gRPC and Protocol Buffer code.

Used for:

* Service communication
* Request and response definitions

---

#### Main Application

```text
main.go
```

Contains:

* Application startup logic
* API handlers
* Business logic

---

## Step 6: Build the Go Application

```dockerfile
RUN CGO_ENABLED=0 GOOS=linux GO111MODULE=on go build -ldflags "-s -w" -o product-catalog main.go
```

### Purpose

Compiles the Go application into a binary executable.

---

### CGO_ENABLED=0

```text
CGO_ENABLED=0
```

Disables CGO support.

### Benefit

Creates a fully static binary.

No external C libraries are required.

---

### GOOS=linux

```text
GOOS=linux
```

Builds the application specifically for Linux.

---

### GO111MODULE=on

```text
GO111MODULE=on
```

Forces Go Modules mode.

Ensures dependency management uses:

```text
go.mod
```

---

### Build Command

```bash
go build
```

Compiles the application.

---

### Build Flags

```text
-ldflags "-s -w"
```

### Purpose

Removes unnecessary debugging information.

Benefits:

* Smaller binary size
* Faster image downloads

---

### Output File

```text
-o product-catalog
```

Creates executable:

```text
product-catalog
```

inside:

```text
/usr/src/app/
```

---

# Stage 2 – Runtime Stage

The second stage creates the final lightweight production image.

---

## Step 7: Use Distroless Runtime Image

```dockerfile
FROM gcr.io/distroless/static-debian12:nonroot
```

### Purpose

Uses a minimal runtime image.

### What is Distroless?

A distroless image contains only:

* Required runtime libraries
* Application binary

It does NOT contain:

* Shell
* Package manager
* Debugging tools

---

### Benefits

* Smaller image size
* Better security
* Reduced attack surface
* Faster deployment

---

### Nonroot

The application runs as a non-root user.

This improves container security.

---

## Step 8: Set Working Directory

```dockerfile
WORKDIR /usr/src/app/
```

### Purpose

Sets the application directory.

Application files will run from:

```text
/usr/src/app/
```

---

## Step 9: Copy Compiled Binary

```dockerfile
COPY --from=builder /usr/src/app/product-catalog/ ./
```

### Purpose

Copies the compiled Go executable from the builder stage.

### Source

```text
/usr/src/app/product-catalog
```

### Destination

```text
/usr/src/app/
```

Result:

```text
/usr/src/app/product-catalog
```

becomes available in the runtime container.

---

## Step 10: Expose Application Port

```dockerfile
EXPOSE ${PRODUCT_CATALOG_PORT}
```

### Purpose

Documents which port the service listens on.

Example:

```bash
PRODUCT_CATALOG_PORT=3550
```

Application listens on:

```text
Port 3550
```

### Note

`EXPOSE` does not publish the port.

Port publishing happens when running the container.

Example:

```bash
docker run -p 3550:3550 product-catalog
```

---

## Step 11: Start the Application

```dockerfile
ENTRYPOINT [ "./product-catalog" ]
```

### Purpose

Defines the command executed when the container starts.

### Executed Command

```bash
./product-catalog
```

This starts the Product Catalog Service.

---

# Multi-Stage Build Flow

```text
Stage 1 (Builder)
│
├── Use Go 1.25 Image
├── Copy go.mod & go.sum
├── Download Dependencies
├── Copy Source Code
├── Compile Application
└── Create product-catalog Binary
          │
          ▼
Stage 2 (Runtime)
│
├── Use Distroless Image
├── Run as Non-Root User
├── Copy Compiled Binary
├── Expose Application Port
└── Start Product Catalog Service
```

---

# Summary

This Dockerfile:

### Build Stage

1. Uses Go 1.25 Bookworm image.
2. Downloads Go dependencies.
3. Copies source code.
4. Compiles the application.
5. Creates a static binary named `product-catalog`.

### Runtime Stage

6. Uses a lightweight Distroless image.
7. Runs as a non-root user.
8. Copies only the compiled binary.
9. Exposes the Product Catalog service port.
10. Starts the application using the `product-catalog` executable.

This approach produces a small, secure, and production-ready Docker image for the Product Catalog Service.
