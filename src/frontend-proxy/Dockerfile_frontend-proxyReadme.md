# Dockerfile Explanation – Frontend Proxy Service (Envoy)

## Overview

This Dockerfile builds and runs the **Frontend Proxy Service** using **Envoy Proxy**.

The Frontend Proxy acts as the entry point for the OpenTelemetry Demo application. It receives requests from users and forwards them to the appropriate backend services.

### Main Responsibilities

* Receive incoming traffic
* Route requests to backend services
* Load balancing
* Service discovery
* Traffic management
* Monitoring and observability

---

# Dockerfile

```dockerfile
FROM envoyproxy/envoy:v1.34-latest

RUN apt-get update && \
    apt-get install -y gettext-base && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

USER envoy

WORKDIR /home/envoy

COPY ./src/frontend-proxy/envoy.tmpl.yaml envoy.tmpl.yaml

EXPOSE ${ENVOY_PORT}
EXPOSE ${ENVOY_ADMIN_PORT}

ENTRYPOINT ["/bin/sh", "-c", "envsubst < envoy.tmpl.yaml > envoy.yaml && envoy -c envoy.yaml;"]
```

---

# Step 1: Use Envoy Base Image

```dockerfile
FROM envoyproxy/envoy:v1.34-latest
```

### Purpose

Downloads the official Envoy Proxy image.

### What is Envoy?

Envoy is a high-performance proxy server used in modern microservice architectures.

### Common Uses

* Reverse Proxy
* API Gateway
* Load Balancer
* Service Mesh

### Benefits

* Fast and scalable
* Supports HTTP, HTTPS, and gRPC
* Traffic routing
* Observability support

---

# Step 2: Install gettext-base

```dockerfile
RUN apt-get update && \
    apt-get install -y gettext-base && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

### Purpose

Installs the package containing the `envsubst` command.

### What is envsubst?

`envsubst` replaces environment variables inside files.

Example:

Template file:

```yaml
port_value: ${ENVOY_PORT}
```

Environment Variable:

```text
ENVOY_PORT=8080
```

Result:

```yaml
port_value: 8080
```

### Why is it needed?

The Envoy configuration contains placeholders that must be replaced with actual values when the container starts.

---

# Step 3: Run as Non-Root User

```dockerfile
USER envoy
```

### Purpose

Runs the container as the `envoy` user instead of the root user.

### Benefits

* Better security
* Reduced privileges
* Kubernetes best practice

---

# Step 4: Set Working Directory

```dockerfile
WORKDIR /home/envoy
```

### Purpose

Sets the current working directory.

All subsequent commands execute inside:

```text
/home/envoy
```
---

# Step 5: Copy Envoy Configuration Template

```dockerfile
COPY ./src/frontend-proxy/envoy.tmpl.yaml envoy.tmpl.yaml
```

### Purpose

Copies the Envoy configuration template into the container.

### Why Template?

The file contains variables such as:

```yaml
port_value: ${ENVOY_PORT}
```

These values are replaced at runtime using `envsubst`.

### Benefit

One Docker image can be used in multiple environments without modifying the configuration file.

---

# Step 6: Expose Application Port

```dockerfile
EXPOSE ${ENVOY_PORT}
```

### Purpose

Documents the main port used by Envoy.

Example:

```text
ENVOY_PORT=8080
```

User traffic enters through this port.

---

# Step 7: Expose Admin Port

```dockerfile
EXPOSE ${ENVOY_ADMIN_PORT}
```

### Purpose

Exposes Envoy's administration interface.

Example:

```text
ENVOY_ADMIN_PORT=9901
```

### Admin Interface Uses

* Metrics
* Statistics
* Health checks
* Configuration inspection

Example URL:

```text
http://localhost:9901/stats
```

---

# Step 8: Start Envoy

```dockerfile
ENTRYPOINT ["/bin/sh", "-c", "envsubst < envoy.tmpl.yaml > envoy.yaml && envoy -c envoy.yaml;"]
```

This is the most important command in the Dockerfile.

---

## Part A: Generate Final Configuration

```bash
envsubst < envoy.tmpl.yaml > envoy.yaml
```

### What Happens?

Template:

```yaml
port_value: ${ENVOY_PORT}
```

Environment Variable:

```text
ENVOY_PORT=8080
```

Generated File:

```yaml
port_value: 8080
```

The final configuration file becomes:

```text
envoy.yaml
```

---

## Part B: Start Envoy

```bash
envoy -c envoy.yaml
```

### What Does It Do?

Starts Envoy using the generated configuration file.

`-c` means:

```text
Configuration File
```

Envoy reads all routing rules and starts listening for incoming traffic.

# Complete Container Startup Flow

```text
Container Starts
        ↓
Read envoy.tmpl.yaml
        ↓
Replace Variables Using envsubst
        ↓
Generate envoy.yaml
        ↓
Start Envoy
        ↓
Listen on ENVOY_PORT
        ↓
Route Requests to Backend Services
```

---

# Important Commands Summary

| Command                            | Purpose                       |
| ---------------------------------- | ----------------------------- |
| FROM envoyproxy/envoy:v1.34-latest | Use Envoy Proxy image         |
| apt-get install gettext-base       | Install envsubst utility      |
| USER envoy                         | Run as non-root user          |
| WORKDIR /home/envoy                | Set working directory         |
| COPY envoy.tmpl.yaml               | Copy configuration template   |
| EXPOSE ${ENVOY_PORT}               | Main traffic port             |
| EXPOSE ${ENVOY_ADMIN_PORT}         | Admin/metrics port            |
| envsubst                           | Replace environment variables |
| envoy -c envoy.yaml                | Start Envoy                   |

---

# Key Learning Points

### Envoy Proxy

A modern reverse proxy and load balancer for microservices.

### Reverse Proxy

Receives requests from clients and forwards them to backend services.

### envsubst

Replaces environment variables inside configuration files.

### Template Configuration

Allows one image to be used across different environments.

### Admin Port

Provides operational metrics and statistics.

### Non-Root User

Improves container security.

---

# Summary

This Dockerfile creates an Envoy Proxy container that serves as the frontend gateway for the OpenTelemetry Demo application.

The container:

1. Uses the official Envoy image.
2. Installs envsubst for variable substitution.
3. Runs as a non-root user.
4. Copies a configuration template.
5. Generates the final Envoy configuration at startup.
6. Starts Envoy and begins routing requests to backend microservices.

The most important command is:

```dockerfile
envsubst < envoy.tmpl.yaml > envoy.yaml && envoy -c envoy.yaml
```

because it dynamically creates the final configuration and starts Envoy using that configuration.
