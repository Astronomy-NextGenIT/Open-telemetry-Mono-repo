# Dockerfile Explanation – Flagd UI Service (Elixir/Phoenix)

## What is this Dockerfile?

This Dockerfile builds and runs the **Flagd UI Service** of the OpenTelemetry Demo.

The service is written in:

* Elixir
* Erlang OTP
* Phoenix Framework

This service provides a web-based UI for managing feature flags.

The Dockerfile follows a **Multi-Stage Build** approach:

### Stage 1: Builder Stage

* Install dependencies
* Compile Elixir code
* Build frontend assets
* Create production release

### Stage 2: Runtime Stage

* Copy release package
* Configure runtime environment
* Start application

---

# Understanding the Technologies

Before understanding the Dockerfile, know these components.

## Elixir

A programming language built on the Erlang VM (BEAM).

Known for:

* High concurrency
* Fault tolerance
* Scalability

---

## Erlang OTP

OTP = Open Telecom Platform

Provides:

* Process management
* Fault recovery
* Distributed computing

Think of OTP as the runtime engine on which Elixir runs.

---

## Phoenix

Phoenix is Elixir's web framework.

Similar to:

```text
Java      → Spring Boot
NodeJS    → Express
Python    → Django
Ruby      → Rails
Elixir    → Phoenix
```

---

# Build Arguments

## Step 1: Define Versions

```dockerfile
ARG ELIXIR_VERSION=1.19.3
ARG OTP_VERSION=28.0.2
ARG DEBIAN_VERSION=bullseye-20251117-slim
```

### What it does

Defines versions used during build.

Example:

```text
Elixir  → 1.19.3
OTP     → 28.0.2
Debian  → bullseye slim
```

---

## Step 2: Create Image Variables

```dockerfile
ARG BUILDER_IMAGE="docker.io/hexpm/elixir:${ELIXIR_VERSION}-erlang-${OTP_VERSION}-debian-${DEBIAN_VERSION}"
ARG RUNNER_IMAGE="docker.io/debian:${DEBIAN_VERSION}"
```

### What it does

Creates reusable image names.

Builder Image:

```text
Elixir + Erlang + Debian
```

Runner Image:

```text
Minimal Debian
```

---

# Builder Stage

## Step 3: Start Builder Stage

```dockerfile
FROM ${BUILDER_IMAGE} AS builder
```

### Purpose

Provides:

* Elixir compiler
* Erlang runtime
* Mix build tool

Needed for compiling the application.

---

## Step 4: Install Build Tools

```dockerfile
RUN apt-get update \
  && apt-get install -y --no-install-recommends build-essential git \
  && rm -rf /var/lib/apt/lists/*
```

### build-essential

Installs:

```text
gcc
g++
make
development tools
```

### git

Required to download dependencies.

### Cleanup

```dockerfile
rm -rf /var/lib/apt/lists/*
```

Removes package cache.

Benefits:

* Smaller image size

---

## Step 5: Create Working Directory

```dockerfile
WORKDIR /app
```

All commands run from:

```text
/app
```

---

# Install Elixir Package Managers

## Step 6: Install Hex and Rebar

```dockerfile
RUN mix local.hex --force \
  && mix local.rebar --force
```

### What is Hex?

Hex is Elixir's package manager.

Equivalent to:

```text
NodeJS  → npm
Python  → pip
Java    → Maven Central
Elixir  → Hex
```

---

### What is Rebar?

Used for Erlang dependency compilation.

---

# Configure Build Environment

## Step 7: Set Production Mode

```dockerfile
ENV MIX_ENV="prod"
```

### What is MIX_ENV?

Elixir environment setting.

Options:

```text
dev
test
prod
```

### Why prod?

Optimized build:

* Better performance
* Smaller release
* Production settings enabled

---

# Dependency Installation

## Step 8: Copy Dependency Files

```dockerfile
COPY ./src/flagd-ui/mix.exs ./src/flagd-ui/mix.lock ./
```

### mix.exs

Equivalent to:

```text
Java      → pom.xml
Gradle    → build.gradle
NodeJS    → package.json
Elixir    → mix.exs
```

Contains:

* Dependencies
* Application settings
* Build configuration

---

### mix.lock

Stores exact dependency versions.

Ensures reproducible builds.

---

## Step 9: Download Dependencies

```dockerfile
RUN mix deps.get --only $MIX_ENV
```

### What it does

Downloads production dependencies.

Example:

```text
Phoenix
Telemetry
HTTP libraries
Database drivers
```

---

## Step 10: Create Config Directory

```dockerfile
RUN mkdir config
```

Creates:

```text
/app/config
```

---

# Configuration Files

## Step 11: Copy Config Files

```dockerfile
COPY ./src/flagd-ui/config/config.exs
COPY ./src/flagd-ui/config/${MIX_ENV}.exs
COPY ./src/flagd-ui/config/runtime.exs
```

### Purpose

Application configuration.

Examples:

```text
Port
Database
Logging
Telemetry
```

---

## Step 12: Compile Dependencies

```dockerfile
RUN mix deps.compile
```

### What it does

Compiles all downloaded dependencies.

Result:

```text
Dependencies Ready For Use
```

---

# Frontend Assets

Phoenix applications contain frontend assets.

Examples:

```text
JavaScript
CSS
Images
```

---

## Step 13: Setup Assets

```dockerfile
RUN mix assets.setup
```

Installs frontend dependencies.

---

## Step 14: Copy Application Files

```dockerfile
COPY ./src/flagd-ui/priv priv
COPY ./src/flagd-ui/lib lib
COPY ./src/flagd-ui/assets assets
```

### priv

Contains:

```text
Static files
Translations
Templates
```

---

### lib

Contains:

```text
Business logic
Controllers
Services
Modules
```

---

### assets

Contains:

```text
CSS
JavaScript
Frontend resources
```

---

## Step 15: Build Frontend Assets

```dockerfile
RUN mix assets.deploy
```

### What it does

Creates optimized frontend files.

Typical actions:

```text
Minify CSS
Minify JS
Bundle assets
```

---

# Build Application

## Step 16: Compile Source Code

```dockerfile
RUN mix compile
```

### What it does

Compiles Elixir application code.

Result:

```text
BEAM bytecode files generated
```

---

## Step 17: Copy Release Configuration

```dockerfile
COPY ./src/flagd-ui/rel rel
```

Contains release settings.

---

## Step 18: Create Production Release

```dockerfile
RUN mix release
```

### What it does

Creates a self-contained production package.

Equivalent to:

```text
.NET     → dotnet publish
Java     → fat JAR
Go       → binary
Elixir   → release
```

Output:

```text
_build/prod/rel/flagd_ui
```

---

# Runtime Stage

## Step 19: Start Runtime Image

```dockerfile
FROM ${RUNNER_IMAGE} AS final
```

Uses lightweight Debian image.

No compiler included.

---

## Step 20: Install Runtime Libraries

```dockerfile
RUN apt-get update \
  && apt-get install -y \
     libstdc++6 \
     openssl \
     libncurses5 \
     locales \
     ca-certificates
```

### Why needed?

The release depends on:

| Package         | Purpose          |
| --------------- | ---------------- |
| openssl         | TLS/HTTPS        |
| libstdc++6      | C++ runtime      |
| libncurses5     | Terminal support |
| locales         | Language support |
| ca-certificates | SSL certificates |

---

# Locale Configuration

## Step 21: Configure UTF-8 Locale

```dockerfile
RUN sed -i '/en_US.UTF-8/s/^# //g' /etc/locale.gen \
  && locale-gen
```

Creates:

```text
en_US.UTF-8
```

locale.

---

## Step 22: Set Locale Variables

```dockerfile
ENV LANG=en_US.UTF-8
ENV LANGUAGE=en_US:en
ENV LC_ALL=en_US.UTF-8
```

Ensures proper text encoding.

---

# Application Setup

## Step 23: Create App Directory

```dockerfile
WORKDIR "/app"
```

---

## Step 24: Set Ownership

```dockerfile
RUN chown nobody /app
```

### Why?

Runs application with non-root permissions.

Security best practice.

---

## Step 25: Set Runtime Environment

```dockerfile
ENV MIX_ENV="prod"
```

Runs application in production mode.

---

## Step 26: Copy Release

```dockerfile
COPY --from=builder --chown=nobody:root \
  /app/_build/${MIX_ENV}/rel/flagd_ui ./
```

### What it does

Copies only the final release package.

Benefits:

```text
No Source Code
No Compiler
No Build Tools
Only Application
```

---

# Networking

## Step 27: Expose Port

```dockerfile
EXPOSE ${FLAGD_UI_PORT}
```

Makes the application accessible.

Example:

```text
FLAGD_UI_PORT=4000
```

---

# Start Application

## Step 28: Start Phoenix Server

```dockerfile
CMD ["sh", "-c", "ulimit -n 65536 && exec /app/bin/server"]
```

### ulimit -n 65536

Sets maximum open file descriptors.

Why?

Web applications may handle:

```text
Many Users
Many Connections
Many Sockets
```

Higher limit prevents connection issues.

---

### exec /app/bin/server

Starts the Phoenix application.

Execution Flow:

```text
Container Starts
        ↓
Production Release Loads
        ↓
Phoenix Server Starts
        ↓
Flagd UI Available
```

---

# Complete Dockerfile Flow

```text
Install Elixir
        ↓
Install Hex & Rebar
        ↓
Download Dependencies
        ↓
Compile Dependencies
        ↓
Build Frontend Assets
        ↓
Compile Application
        ↓
Create Production Release
        ↓
Create Runtime Image
        ↓
Install Runtime Libraries
        ↓
Copy Release
        ↓
Start Phoenix Server
```

# Summary

This Dockerfile builds and deploys the Flagd UI service using Elixir and Phoenix. It creates a production release and packages only the runtime artifacts into the final image.

Benefits:

* Multi-stage build
* Smaller final image
* Production-optimized release
* Secure non-root execution
* Compiled frontend assets
* High concurrency through Erlang/OTP
* Efficient resource usage
* Suitable for Kubernetes deployments
