# Dockerfile Explanation – Accounting Service

## What is this Dockerfile?

A Dockerfile is a set of instructions used to build a Docker image for an application.

This Dockerfile builds and runs the Accounting microservice written in .NET and prepares it for OpenTelemetry monitoring.

---

# Step 1: Use .NET SDK Image for Building

```dockerfile
FROM --platform=${BUILDPLATFORM} mcr.microsoft.com/dotnet/sdk:10.0 AS builder
```

### What it does

* Downloads the .NET 10 SDK image.
* SDK image contains all tools required to compile the application.
* Creates a build stage named **builder**.

### Why it is needed

The application source code must be compiled before it can run.

---

# Step 2: Define Build Arguments

```dockerfile
ARG TARGETARCH
ARG BUILD_CONFIGURATION=Release
```

### What it does

Creates variables that can be used during the build.

### Variables

| Variable            | Purpose                                      |
| ------------------- | -------------------------------------------- |
| TARGETARCH          | Target CPU architecture (amd64, arm64, etc.) |
| BUILD_CONFIGURATION | Build mode (Release or Debug)                |

### Why it is needed

Allows the same Dockerfile to build images for different platforms.

---

# Step 3: Set Working Directory

```dockerfile
WORKDIR /src
```

### What it does

Moves into the `/src` directory inside the container.

### Why it is needed

All following commands will execute from this location.

---

# Step 4: Copy Application Source Code

```dockerfile
COPY ["/src/accounting/", "Accounting/"]
```

### What it does

Copies the Accounting service source code from the repository into the container.

### Result

```text
/src
 └── Accounting
```

---

# Step 5: Copy Proto File

```dockerfile
COPY ["/pb/demo.proto", "Accounting/src/protos/demo.proto"]
```

### What it does

Copies the gRPC Protocol Buffer file.

### Why it is needed

This file defines the communication structure used between microservices.

---

# Step 6: Restore Dependencies

```dockerfile
RUN dotnet restore "./Accounting/Accounting.csproj" -r linux-$TARGETARCH
```

### What it does

Downloads all required .NET packages and dependencies.

### Similar to

Running:

```bash
dotnet restore
```

on your local machine.

### Why it is needed

The application cannot be built without its required libraries.

---

# Step 7: Move to Project Folder

```dockerfile
WORKDIR "/src/Accounting"
```

### What it does

Changes the current directory to the Accounting project folder.

---

# Step 8: Build the Application

```dockerfile
RUN dotnet build "./Accounting.csproj" -r linux-$TARGETARCH -c $BUILD_CONFIGURATION -o /app/build
```

### What it does

Compiles the source code.

### Parameters

| Option | Meaning            |
| ------ | ------------------ |
| -r     | Runtime platform   |
| -c     | Release/Debug mode |
| -o     | Output directory   |

### Output

Compiled files are stored in:

```text
/app/build
```

---

# Step 9: Create Publish Stage

```dockerfile
FROM builder AS publish
```

### What it does

Creates another build stage called **publish**.

### Why it is needed

Separates the publishing process from the build process.

---

# Step 10: Publish the Application

```dockerfile
RUN dotnet publish "./Accounting.csproj" -r linux-$TARGETARCH -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false
```

### What it does

Creates optimized production-ready files.

### Output

```text
/app/publish
```

### Why it is needed

Publish creates only the files required to run the application.

---

# Step 11: Use Lightweight Runtime Image

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:10.0
```

### What it does

Uses the smaller ASP.NET runtime image.

### Why it is needed

The SDK image is large.

The runtime image contains only what is needed to run the application.

This reduces image size significantly.

---

# Step 12: Switch to App User

```dockerfile
USER app
```

### What it does

Runs the container using a non-root user.

### Why it is needed

Improves security.

---

# Step 13: Set Application Directory

```dockerfile
WORKDIR /app
```

### What it does

Sets `/app` as the working directory.

---

# Step 14: Copy Published Files

```dockerfile
COPY --from=publish /app/publish .
```

### What it does

Copies the published application files from the publish stage.

### Result

Application files are available inside:

```text
/app
```

---

# Step 15: Switch to Root User

```dockerfile
USER root
```

### What it does

Temporarily switches to root.

### Why it is needed

Root permissions are required to create folders and modify ownership.

---

# Step 16: Create Log Directory

```dockerfile
RUN mkdir -p "/var/log/opentelemetry/dotnet"
```

### What it does

Creates a directory where OpenTelemetry logs can be stored.

---

# Step 17: Give Permission to App User

```dockerfile
RUN chown app "/var/log/opentelemetry/dotnet"
```

### What it does

Makes the app user the owner of the log directory.

### Why it is needed

Allows the application to write logs.

---

# Step 18: Give Permission to Startup Script

```dockerfile
RUN chown app "/app/instrument.sh"
```

### What it does

Allows the app user to execute the startup script.

---

# Step 19: Return to App User

```dockerfile
USER app
```

### What it does

Switches back to the non-root user for security.

---

# Step 20: Set OpenTelemetry Environment Variable

```dockerfile
ENV OTEL_DOTNET_AUTO_TRACES_ADDITIONAL_SOURCES=Accounting.Consumer
```

### What it does

Configures OpenTelemetry tracing.

### Why it is needed

Allows traces generated by `Accounting.Consumer` to be collected and sent to OpenTelemetry.

---

# Step 21: Start the Application

```dockerfile
ENTRYPOINT ["./instrument.sh", "dotnet", "Accounting.dll"]
```

### What it does

Runs the startup script and then launches the application.

### Execution Flow

```text
Container Starts
       ↓
instrument.sh Executes
       ↓
OpenTelemetry Instrumentation Starts
       ↓
dotnet Accounting.dll Runs
       ↓
Accounting Service Becomes Available
```

---

# Overall Flow of the Dockerfile

```text
Source Code
     ↓
Copy Files
     ↓
Restore Dependencies
     ↓
Build Application
     ↓
Publish Application
     ↓
Create Runtime Image
     ↓
Copy Published Files
     ↓
Configure OpenTelemetry
     ↓
Start Accounting Service
```

## Summary

This Dockerfile follows a Multi-Stage Build approach:

1. Build the .NET application.
2. Publish optimized production files.
3. Create a lightweight runtime image.
4. Configure OpenTelemetry monitoring.
5. Run the Accounting microservice securely using a non-root user.

Benefits:

* Smaller image size
* Better security
* Faster deployments
* Built-in OpenTelemetry observability
* Multi-platform support (amd64/arm64)
