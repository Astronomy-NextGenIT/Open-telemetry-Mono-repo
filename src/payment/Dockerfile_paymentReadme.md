# Dockerfile Explanation – Payment Service

This Dockerfile is used to build and run the **Payment Service**, which is a Node.js application. It uses a multi-stage build to create a smaller and more secure Docker image by separating the dependency installation stage from the runtime stage.

---

# Complete Dockerfile

```dockerfile
FROM docker.io/library/node:22-slim AS builder

WORKDIR /usr/src/app/

COPY ./src/payment/package.json package.json
COPY ./src/payment/package-lock.json package-lock.json

RUN npm ci --omit=dev

# -----------------------------------------------------------------------------

FROM gcr.io/distroless/nodejs22-debian12:nonroot

WORKDIR /usr/src/app/

COPY --from=builder /usr/src/app/node_modules/ node_modules/

COPY ./pb/demo.proto demo.proto

COPY ./src/payment/charge.js charge.js
COPY ./src/payment/index.js index.js
COPY ./src/payment/logger.js logger.js
COPY ./src/payment/opentelemetry.js opentelemetry.js

EXPOSE ${PAYMENT_PORT}

CMD ["--require=./opentelemetry.js", "index.js"]
```

---

# Step-by-Step Explanation

## 1. Builder Stage

```dockerfile
FROM docker.io/library/node:22-slim AS builder
```

### Purpose

Uses Node.js 22 Slim image as the build environment.

### Why?

This image contains:

* Node.js runtime
* npm package manager
* Required build tools

The stage is named:

```text
builder
```

and is used only for installing dependencies.

---

## 2. Set Working Directory

```dockerfile
WORKDIR /usr/src/app/
```

### Purpose

Creates and sets the working directory.

All following commands run from:

```text
/usr/src/app/
```

---

## 3. Copy Package Files

```dockerfile
COPY ./src/payment/package.json package.json
COPY ./src/payment/package-lock.json package-lock.json
```

### Purpose

Copies Node.js dependency files.

### package.json

Contains:

* Application name
* Version
* Dependencies
* Scripts

### package-lock.json

Contains:

* Exact dependency versions
* Dependency tree information

This ensures consistent installations.

---

## 4. Install Dependencies

```dockerfile
RUN npm ci --omit=dev
```

### Purpose

Installs application dependencies.

### Why Use npm ci?

Benefits:

* Faster than npm install
* Uses package-lock.json exactly
* Provides reproducible builds

### Option Used

```text
--omit=dev
```

This skips development dependencies.

Examples of skipped packages:

* Testing frameworks
* Linters
* Development tools

### Benefit

Smaller production image.

---

# Runtime Stage

## 5. Use Distroless Runtime Image

```dockerfile
FROM gcr.io/distroless/nodejs22-debian12:nonroot
```

### Purpose

Creates the final runtime image.

### What is Distroless?

A distroless image contains only:

* Node.js runtime
* Required libraries

It does NOT contain:

* Shell
* Package manager
* Extra Linux utilities

### Benefits

* Smaller image size
* Better security
* Fewer vulnerabilities

### Nonroot

The container runs as a non-root user automatically.

---

## 6. Set Working Directory

```dockerfile
WORKDIR /usr/src/app/
```

### Purpose

Sets the application directory.

Application files will be stored in:

```text
/usr/src/app/
```

---

## 7. Copy Installed Dependencies

```dockerfile
COPY --from=builder /usr/src/app/node_modules/ node_modules/
```

### Purpose

Copies Node.js packages from the builder stage.

### Result

```text
/usr/src/app/node_modules/
```

contains all required production dependencies.

### Why?

Avoids reinstalling packages in the runtime image.

---

## 8. Copy Protocol Buffer File

```dockerfile
COPY ./pb/demo.proto demo.proto
```

### Purpose

Copies the Protocol Buffer definition file.

### What is demo.proto?

Defines message formats used in gRPC communication.

Example:

```text
PaymentRequest
PaymentResponse
```

This file helps services communicate using gRPC.

---

## 9. Copy Application Files

### Charge Logic

```dockerfile
COPY ./src/payment/charge.js charge.js
```

Contains payment processing logic.

Example:

* Credit card charging
* Payment validation
* Transaction handling

---

### Main Application

```dockerfile
COPY ./src/payment/index.js index.js
```

This is the main application entry point.

Responsible for:

* Starting the service
* Listening for requests
* Initializing components

---

### Logging Module

```dockerfile
COPY ./src/payment/logger.js logger.js
```

Handles application logging.

Examples:

* Information logs
* Warning logs
* Error logs

---

### OpenTelemetry Configuration

```dockerfile
COPY ./src/payment/opentelemetry.js opentelemetry.js
```

Configures OpenTelemetry monitoring.

Used for collecting:

* Traces
* Metrics
* Logs

---

## 10. Expose Application Port

```dockerfile
EXPOSE ${PAYMENT_PORT}
```

### Purpose

Documents which port the service uses.

Example:

```bash
PAYMENT_PORT=8080
```

The application will listen on:

```text
Port 8080
```

### Note

EXPOSE does not publish the port.

Port mapping happens when running the container.

Example:

```bash
docker run -p 8080:8080 payment-service
```

---

## 11. Start the Application

```dockerfile
CMD ["--require=./opentelemetry.js", "index.js"]
```

### Purpose

Defines the command that runs when the container starts.

### What Happens?

Node.js starts the application and first loads:

```text
opentelemetry.js
```

Then it starts:

```text
index.js
```

### Execution Flow

```text
Load OpenTelemetry Configuration
            │
            ▼
Initialize Monitoring
            │
            ▼
Start Payment Service
            │
            ▼
Accept Requests
```

---

# Multi-Stage Build Flow

```text
Builder Stage
│
├── Use Node.js 22 Slim
├── Copy package files
├── Install dependencies
└── Store node_modules
         │
         ▼
Runtime Stage
│
├── Use Distroless Image
├── Copy node_modules
├── Copy application files
├── Copy demo.proto
├── Configure OpenTelemetry
└── Start Payment Service
```

---

# Summary

This Dockerfile:

1. Uses Node.js 22 Slim for building dependencies.
2. Installs only production dependencies.
3. Uses a secure Distroless runtime image.
4. Runs as a non-root user.
5. Copies gRPC protocol definitions.
6. Copies payment service source code.
7. Configures OpenTelemetry monitoring.
8. Exposes the payment service port.
9. Starts the application using `index.js`.
10. Loads OpenTelemetry before the application starts so that traces, metrics, and logs are automatically collected.
