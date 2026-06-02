# Dockerfile Explanation – Ad Service (Java)

## What is this Dockerfile?

This Dockerfile is used to build and run the Ad Service of the OpenTelemetry Demo application.

The service is developed using Java and Gradle. It also includes OpenTelemetry Java Agent for automatic monitoring and tracing.

The Dockerfile uses a Multi-Stage Build approach to keep the final image small and efficient.

---

# Step 1: Use Java JDK Image for Building

```dockerfile
FROM --platform=${BUILDPLATFORM} eclipse-temurin:21-jdk AS builder
```

### What it does

* Downloads Eclipse Temurin Java 21 JDK image.
* JDK (Java Development Kit) contains tools required to compile Java applications.
* Creates a build stage named **builder**.

### Why it is needed

The application source code must be compiled before it can run.

---

# Step 2: Set Working Directory

```dockerfile
WORKDIR /usr/src/app/
```

### What it does

Sets the working directory inside the container.

All upcoming commands will execute from this location.

---

# Step 3: Copy Gradle Files

```dockerfile
COPY ./src/ad/gradlew* ./src/ad/settings.gradle* ./src/ad/build.gradle ./
COPY ./src/ad/gradle ./gradle
```

### What it does

Copies Gradle-related files:

* gradlew
* settings.gradle
* build.gradle
* gradle folder

### Why it is needed

These files contain project configuration and dependency information.

---

# Step 4: Make Gradle Executable

```dockerfile
RUN chmod +x ./gradlew
```

### What it does

Gives execute permission to the Gradle Wrapper script.

### Why it is needed

Without execute permission, Gradle commands cannot run.

---

# Step 5: Initialize Gradle

```dockerfile
RUN ./gradlew
```

### What it does

Runs Gradle Wrapper.

### Why it is needed

Downloads the required Gradle version and prepares the build environment.

---

# Step 6: Download Dependencies

```dockerfile
RUN ./gradlew downloadRepos
```

### What it does

Downloads all required Java libraries and dependencies.

### Why it is needed

The application requires external libraries to compile and run.

---

# Step 7: Copy Application Source Code

```dockerfile
COPY ./src/ad/ ./
```

### What it does

Copies the complete Ad Service source code into the container.

---

# Step 8: Copy Proto Files

```dockerfile
COPY ./pb/ ./proto
```

### What it does

Copies Protocol Buffer files.

### Why it is needed

Proto files define communication contracts between microservices using gRPC.

---

# Step 9: Make Gradle Executable Again

```dockerfile
RUN chmod +x ./gradlew
```

### What it does

Ensures the Gradle wrapper still has execute permission after files are copied.

---

# Step 10: Build and Install Application

```dockerfile
RUN ./gradlew installDist -PprotoSourceDir=./proto
```

### What it does

Builds the application and creates a distributable package.

### Parameter

| Parameter                | Purpose                    |
| ------------------------ | -------------------------- |
| -PprotoSourceDir=./proto | Location of protobuf files |

### Output

Gradle creates application binaries and scripts inside:

```text
build/install/
```

---

# Step 11: Start Runtime Stage

```dockerfile
FROM eclipse-temurin:21-jre
```

### What it does

Uses Java Runtime Environment (JRE) image.

### Why it is needed

JRE contains only the components needed to run Java applications.

It is much smaller than the JDK image.

---

# Step 12: Define OpenTelemetry Agent Version

```dockerfile
ARG OTEL_JAVA_AGENT_VERSION
```

### What it does

Creates a variable that stores the OpenTelemetry Java Agent version.

Example:

```text
2.15.0
```

---

# Step 13: Set Runtime Working Directory

```dockerfile
WORKDIR /usr/src/app/
```

### What it does

Sets the runtime working directory.

---

# Step 14: Copy Built Application

```dockerfile
COPY --from=builder /usr/src/app/ ./
```

### What it does

Copies the built application from the builder stage into the runtime image.

### Why it is needed

The runtime image only needs the final application files.

---

# Step 15: Download OpenTelemetry Java Agent

```dockerfile
ADD --chmod=644 https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases/download/v$OTEL_JAVA_AGENT_VERSION/opentelemetry-javaagent.jar /usr/src/app/opentelemetry-javaagent.jar
```

### What it does

Downloads the OpenTelemetry Java Agent JAR file.

### Why it is needed

The Java Agent automatically collects:

* Traces
* Metrics
* Logs

without changing application code.

### Result

```text
/usr/src/app/opentelemetry-javaagent.jar
```

---

# Step 16: Configure Java Agent

```dockerfile
ENV JAVA_TOOL_OPTIONS="-javaagent:/usr/src/app/opentelemetry-javaagent.jar -Xmx200m"
```

### What it does

Sets Java startup options.

### Parameters

| Option     | Purpose                        |
| ---------- | ------------------------------ |
| -javaagent | Loads OpenTelemetry Java Agent |
| -Xmx200m   | Limits JVM memory to 200 MB    |

### Why it is needed

Ensures monitoring starts automatically when the application launches.

---

# Step 17: Expose Application Port

```dockerfile
EXPOSE ${AD_PORT}
```

### What it does

Makes the Ad Service port available outside the container.

### Example

```text
AD_PORT=8080
```

### Why it is needed

Allows users and other services to access the application.

---

# Step 18: Start the Application

```dockerfile
ENTRYPOINT [ "./build/install/opentelemetry-demo-ad/bin/Ad" ]
```

### What it does

Starts the Ad Service.

### Execution Flow

```text
Container Starts
       ↓
Java Runtime Starts
       ↓
OpenTelemetry Java Agent Loads
       ↓
Ad Service Starts
       ↓
Service Accepts Requests
```

---

# Overall Flow of the Dockerfile

```text
Copy Gradle Files
        ↓
Download Dependencies
        ↓
Copy Source Code
        ↓
Copy Proto Files
        ↓
Build Application
        ↓
Create Runtime Image
        ↓
Copy Built Files
        ↓
Download OpenTelemetry Agent
        ↓
Configure Monitoring
        ↓
Start Ad Service
```

## Summary

This Dockerfile builds and runs the Java-based Ad Service using Gradle and OpenTelemetry.

Main Benefits:

* Multi-stage build reduces image size.
* Uses Java 21.
* Automatically downloads dependencies.
* Supports gRPC through protobuf files.
* Includes OpenTelemetry Java Agent.
* Collects traces and metrics automatically.
* Uses lightweight JRE image for production.
* Optimized for containerized deployments.
