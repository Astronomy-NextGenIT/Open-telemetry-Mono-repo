# Dockerfile Explanation – Fraud Detection Service (Java)

## What is this Dockerfile?

This Dockerfile builds and runs the **Fraud Detection Service** of the OpenTelemetry Demo application.

The service is written in **Java** and built using **Gradle**.

Its main responsibilities are:

1. Compile Java source code.
2. Build a deployable JAR file.
3. Add OpenTelemetry Java Agent.
4. Run the application in a lightweight production container.

This Dockerfile uses a **Multi-Stage Build** approach.

---

# High-Level Flow

```text
Source Code
      ↓
Copy Proto Files
      ↓
Gradle Build
      ↓
Create Fat JAR
      ↓
Create Runtime Image
      ↓
Download OpenTelemetry Agent
      ↓
Start Fraud Detection Service
```

---

# Understanding Important Concepts

Before understanding the Dockerfile, know these concepts.

## Gradle

Gradle is Java's build tool.

Similar to:

```text
NodeJS   → npm
Python   → pip
Ruby     → Bundler
Java     → Gradle
```

Gradle performs:

* Dependency download
* Compilation
* Packaging
* Testing
* Deployment

---

## JAR File

JAR = Java Archive

A JAR contains:

```text
Compiled Java Classes
Libraries
Resources
Configuration Files
```

Think of it as a ZIP file for Java applications.

---

## Fat JAR (Uber JAR)

A normal JAR may require external libraries.

A Fat JAR contains:

```text
Application Code
+
All Dependencies
+
Required Libraries
```

Everything is bundled into one file.

Example:

```text
fraud-detection-1.0-all.jar
```

The word **all** indicates a Fat JAR.

---

# Builder Stage

## Step 1: Use Gradle Builder Image

```dockerfile
FROM --platform=${BUILDPLATFORM} gradle:8-jdk17 AS builder
```

### What it does

Downloads:

```text
Gradle 8
Java JDK 17
```

### Why JDK?

JDK contains:

```text
Java Compiler
Build Tools
Gradle Support
```

Needed to compile Java code.

---

## Step 2: Set Working Directory

```dockerfile
WORKDIR /usr/src/app/
```

Creates:

```text
/usr/src/app
```

All future commands execute from this directory.

---

## Step 3: Copy Fraud Detection Source Code

```dockerfile
COPY ./src/fraud-detection/ ./
```

### What it copies

Typically:

```text
build.gradle
settings.gradle
src/
```

and other project files.

---

## Step 4: Copy Shared Proto Files

```dockerfile
COPY ./pb/ ./src/main/proto/
```

### What are Proto Files?

Proto files define message structures used by gRPC.

Example:

```text
Fraud Service
      ↓
Receives Order Data
      ↓
Checks Fraud Risk
      ↓
Returns Response
```

All services understand the same data format through protobuf.

---

## Step 5: Build Application

```dockerfile
RUN gradle shadowJar
```

This is the most important build command.

---

### What is shadowJar?

The Shadow Plugin creates a Fat JAR.

Normal build:

```text
Application.jar
Dependencies Separate
```

Shadow build:

```text
Application.jar
+
All Dependencies
=
Single Fat JAR
```

Result:

```text
build/libs/fraud-detection-1.0-all.jar
```

---

### Why Use shadowJar?

Benefits:

* Easier deployment
* No dependency issues
* Single executable file
* Container friendly

---

# Runtime Stage

After building the application, Docker starts a new stage.

---

## Step 6: Use Distroless Java Image

```dockerfile
FROM gcr.io/distroless/java17-debian12:nonroot
```

### What is Distroless?

Distroless images contain only:

```text
Java Runtime
Required Libraries
Application
```

They do NOT contain:

```text
Shell
Package Manager
Compiler
Build Tools
```

---

### Benefits

#### Smaller Image

Less storage usage.

#### Better Security

Fewer installed packages.

#### Faster Deployment

Smaller image downloads faster.

---

### What Does nonroot Mean?

```dockerfile
:nonroot
```

Application runs as a non-root user.

Benefits:

```text
Improved Security
Reduced Privileges
Kubernetes Best Practice
```

---

# OpenTelemetry Integration

## Step 7: Define Agent Version

```dockerfile
ARG OTEL_JAVA_AGENT_VERSION
```

### What it does

Creates a variable.

Example:

```text
2.15.0
```

Used when downloading OpenTelemetry Agent.

---

## Step 8: Set Working Directory

```dockerfile
WORKDIR /usr/src/app/
```

Application files will be stored here.

---

## Step 9: Download OpenTelemetry Java Agent

```dockerfile
ADD --chmod=644 https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases/download/v$OTEL_JAVA_AGENT_VERSION/opentelemetry-javaagent.jar /usr/src/app/opentelemetry-javaagent.jar
```

### What is OpenTelemetry Java Agent?

A special Java library that automatically collects:

```text
Traces
Metrics
Logs
```

without changing application code.

---

### Why is it useful?

Without agent:

```text
Application
      ↓
Manual Instrumentation Required
```

With agent:

```text
Application
      ↓
Agent Automatically Collects Telemetry
```

---

## Step 10: Configure Java Agent

```dockerfile
ENV JAVA_TOOL_OPTIONS="-javaagent:/usr/src/app/opentelemetry-javaagent.jar -Xmx180m"
```

### What is JAVA_TOOL_OPTIONS?

A standard Java environment variable.

Automatically applied when Java starts.

---

### -javaagent

```text
Loads OpenTelemetry Agent
```

---

### -Xmx180m

```text
Maximum Heap Memory = 180 MB
```

Limits memory usage.

Benefits:

```text
Predictable Resource Usage
Better Kubernetes Scheduling
```

---

# Copy Application

## Step 11: Copy Fat JAR

```dockerfile
COPY --from=builder /usr/src/app/build/libs/fraud-detection-1.0-all.jar fraud-detection-1.0-all.jar
```

### What it does

Copies the generated Fat JAR from builder stage.

Result:

```text
/usr/src/app/
 ├── fraud-detection-1.0-all.jar
 └── opentelemetry-javaagent.jar
```

---

# Start Application

## Step 12: Run Fraud Detection Service

```dockerfile
ENTRYPOINT [ "java", "-jar", "fraud-detection-1.0-all.jar" ]
```

### What it does

Starts Java application.

Equivalent command:

```bash
java -jar fraud-detection-1.0-all.jar
```

---

# Complete Dockerfile Flow

```text
Copy Source Code
        ↓
Copy Proto Files
        ↓
Gradle shadowJar
        ↓
Create Fat JAR
        ↓
Create Distroless Runtime
        ↓
Download OpenTelemetry Agent
        ↓
Configure Java Agent
        ↓
Copy Fat JAR
        ↓
Start Application
```

# Key Learning Points

## Gradle

Java build automation tool.

## shadowJar

Creates a single executable Fat JAR.

## Fat JAR

Contains application code and all dependencies.

## Distroless Image

Minimal runtime image with better security.

## OpenTelemetry Java Agent

Automatically captures telemetry.

## JAVA_TOOL_OPTIONS

Applies JVM startup parameters.

## -Xmx180m

Limits Java heap memory.

## ENTRYPOINT

Defines container startup command.

# Summary

This Dockerfile builds the Fraud Detection Service using Gradle and packages it as a Fat JAR. It then runs the application in a lightweight Distroless container with automatic OpenTelemetry instrumentation.

Major Benefits:

* Multi-stage build
* Single deployable JAR
* Small runtime image
* Distroless security model
* Automatic OpenTelemetry monitoring
* Controlled memory usage
* Production-ready deployment
* Kubernetes friendly
