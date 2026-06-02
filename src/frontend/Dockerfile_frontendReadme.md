# Dockerfile Explanation – Frontend Service (Next.js / Node.js)

## Overview

This Dockerfile builds and runs the **Frontend Service** of the OpenTelemetry Demo application.

The frontend is built using:

* Node.js
* Next.js
* TypeScript
* OpenTelemetry Instrumentation

This Dockerfile uses a **3-Stage Build** approach:

### Stage 1 – Builder

* Install all dependencies
* Compile Next.js application
* Generate production build

### Stage 2 – Dependencies

* Install only production dependencies

### Stage 3 – Runtime

* Run the application using a lightweight Distroless image

---

# High-Level Flow

```text
Frontend Source Code
          ↓
Install Dependencies
          ↓
Build Next.js Application
          ↓
Install Production Dependencies
          ↓
Create Runtime Image
          ↓
Copy Build Artifacts
          ↓
Load OpenTelemetry Instrumentation
          ↓
Start Frontend Server
```
---

# Stage 1: Builder Stage

## Step 1: Use Node.js Builder Image

```dockerfile
FROM docker.io/library/node:24-slim AS builder
```

### Purpose

Downloads Node.js 24 image.

Contains:

* Node.js Runtime
* npm
* Build tools

### Why Builder Stage?

Used to compile the application.

---

## Step 2: Create Working Directory

```dockerfile
WORKDIR /app
```

Creates:

```text
/app
```

All future commands execute from this location.

---

## Step 3: Copy Package Files

```dockerfile
COPY ./src/frontend/package.json package.json
COPY ./src/frontend/package-lock.json package-lock.json
```

### package.json

Equivalent to:

```text
Java      → pom.xml
Gradle    → build.gradle
Python    → requirements.txt
Node.js   → package.json
```

Contains:

* Dependencies
* Scripts
* Project metadata

---

### package-lock.json

Contains exact dependency versions.

Benefits:

* Reproducible builds
* Consistent environments

---

## Step 4: Install Dependencies

```dockerfile
RUN npm ci
```

### What is npm ci?

Installs dependencies from package-lock.json.

### Difference Between npm install and npm ci

| npm install          | npm ci                 |
| -------------------- | ---------------------- |
| Slower               | Faster                 |
| Can modify lock file | Uses lock file exactly |
| Development use      | CI/CD use              |

### Why npm ci?

Provides reliable builds in CI/CD pipelines.

---

# Step 5: Copy Application Code

Several COPY commands move source code into the container.

### Components

```dockerfile
COPY ./src/frontend/components/ components/
```

Reusable UI components.

Examples:

```text
Buttons
Cards
Forms
Navigation Bars
```

---

### Pages

```dockerfile
COPY ./src/frontend/pages/ pages/
```

Contains website pages.

Examples:

```text
Home Page
Product Page
Cart Page
Checkout Page
```

---

### Services

```dockerfile
COPY ./src/frontend/services/ services/
```

Handles communication with backend APIs.

---

### Gateways

```dockerfile
COPY ./src/frontend/gateways/ gateways/
```

Connects frontend to backend services.

---

### Providers

```dockerfile
COPY ./src/frontend/providers/ providers/
```

Provides shared application state.

Examples:

```text
Authentication
Theme
Context Providers
```

---

### Styles

```dockerfile
COPY ./src/frontend/styles/ styles/
```

Contains CSS and styling files.

---

### Types

```dockerfile
COPY ./src/frontend/types/ types/
```

Contains TypeScript type definitions.

---

### Protos

```dockerfile
COPY ./src/frontend/protos/ protos/
```

Contains Protocol Buffer definitions used for communication.

---

### Utility Files

```dockerfile
COPY ./src/frontend/utils/
```

Contains helper functions.

Examples:

```text
Telemetry
API Requests
Image Loading
Enums
```

---

## Step 6: Copy Configuration Files

```dockerfile
COPY ./src/frontend/next.config.js next.config.js
COPY ./src/frontend/tsconfig.json tsconfig.json
```

### next.config.js

Configures Next.js.

Examples:

```text
Routing
Build Settings
Optimization
```

---

### tsconfig.json

Configures TypeScript.

Examples:

```text
Compiler Options
Path Aliases
Type Checking Rules
```

---

## Step 7: Build Application

```dockerfile
RUN npm run build
```

### What Happens?

Next.js creates an optimized production build.

Generated Output:

```text
.next/
```

Contains:

```text
Compiled Pages
Static Assets
Server Files
Optimized JavaScript
```

---

# Stage 2: Production Dependencies

## Step 8: Create Dependency Stage

```dockerfile
FROM docker.io/library/node:24-slim AS deps
```

Purpose:

Install only production dependencies.

---

## Step 9: Copy Package Files

```dockerfile
COPY ./src/frontend/package.json package.json
COPY ./src/frontend/package-lock.json package-lock.json
```

---

## Step 10: Install Production Dependencies

```dockerfile
RUN npm ci --omit=dev
```

### What does --omit=dev do?

Installs only production packages.

Skips:

```text
Testing Tools
Linters
Development Libraries
```

Benefits:

* Smaller image
* Faster startup
* Better security

---

# Stage 3: Runtime Stage

## Step 11: Use Distroless Node.js Image

```dockerfile
FROM gcr.io/distroless/nodejs24-debian13:nonroot
```

### What is Distroless?

Contains only:

```text
Node.js Runtime
Required Libraries
Application
```

Does NOT contain:

```text
Shell
Package Managers
Build Tools
Compilers
```

---

### Benefits

#### Smaller Image

Less storage usage.

#### Better Security

Fewer packages means fewer vulnerabilities.

#### Faster Deployments

Smaller image downloads faster.

---

### nonroot

Runs the application as a non-root user.

Security best practice.

---

## Step 12: Create Working Directory

```dockerfile
WORKDIR /app
```

---

## Step 13: Copy Next.js Build Output

```dockerfile
COPY --from=builder /app/.next/standalone/ ./
```

### What is standalone?

A self-contained production version of the application.

Contains:

```text
Compiled Server
Required Runtime Files
```

---

## Step 14: Copy Static Files

```dockerfile
COPY --from=builder /app/.next/static/ .next/static/
```

Contains:

```text
Images
CSS
JavaScript
Fonts
```

---

## Step 15: Copy Production Dependencies

```dockerfile
COPY --from=deps /app/node_modules/ node_modules/
```

Copies only production packages.

---

## Step 16: Copy Public Folder

```dockerfile
COPY ./src/frontend/public/ public/
```

Contains:

```text
Images
Icons
Static Content
```

---

## Step 17: Copy OpenTelemetry Instrumentation

```dockerfile
COPY ./src/frontend/utils/telemetry/Instrumentation.js Instrumentation.js
```

### Purpose

Loads OpenTelemetry instrumentation.

Used to collect:

```text
Traces
Metrics
Logs
```

from the frontend application.

---

# Networking

## Step 18: Expose Application Port

```dockerfile
EXPOSE ${FRONTEND_PORT}
```

Example:

```text
FRONTEND_PORT=8080
```

Allows users to access the frontend application.

---

# Start Application

## Step 19: Run Frontend Server

```dockerfile
CMD ["--require=./Instrumentation.js", "server.js"]
```

### What Happens?

Node.js starts.

Before the application loads:

```text
Instrumentation.js
```

is loaded.

This enables OpenTelemetry monitoring.

After that:

```text
server.js
```

starts the Next.js application.

---

# Startup Flow

```text
Container Starts
        ↓
Load Instrumentation.js
        ↓
Initialize OpenTelemetry
        ↓
Start Node.js Server
        ↓
Serve Frontend Application
        ↓
Send Telemetry Data
```

---

# Complete Dockerfile Flow

```text
Install Dependencies
        ↓
Copy Source Code
        ↓
Build Next.js Application
        ↓
Install Production Dependencies
        ↓
Create Distroless Runtime
        ↓
Copy Build Output
        ↓
Copy node_modules
        ↓
Copy Public Assets
        ↓
Load OpenTelemetry Instrumentation
        ↓
Start Frontend Server
```

---

# Important Commands Summary

| Command                                             | Purpose                      |
| --------------------------------------------------- | ---------------------------- |
| FROM node:24-slim                                   | Builder image                |
| npm ci                                              | Install dependencies         |
| npm run build                                       | Build Next.js application    |
| npm ci --omit=dev                                   | Install production packages  |
| FROM distroless/nodejs24                            | Runtime image                |
| COPY .next/standalone                               | Copy compiled application    |
| COPY node_modules                                   | Copy production dependencies |
| COPY Instrumentation.js                             | Enable telemetry             |
| EXPOSE ${FRONTEND_PORT}                             | Application port             |
| CMD ["--require=./Instrumentation.js", "server.js"] | Start application            |

---

# Key Learning Points

### Next.js

React framework used for building web applications.

### npm ci

Reliable dependency installation for CI/CD pipelines.

### Standalone Build

Self-contained Next.js production build.

### Distroless Image

Minimal runtime image with improved security.

### Production Dependencies

Only runtime packages are included.

### OpenTelemetry Instrumentation

Collects traces, metrics, and logs automatically.

### Multi-Stage Build

Separates build environment from runtime environment.

---

# Summary

This Dockerfile builds and deploys the Frontend Service using Next.js and Node.js. It uses a three-stage build process to create a secure, optimized, 
and lightweight production image.

Benefits:

* Faster builds
* Smaller image size
* Improved security
* Production-only dependencies
* OpenTelemetry integration
* Optimized Next.js build
* Non-root execution
* Kubernetes-friendly deployment
