# Valkey Deployment (`deploy.yaml`) Documentation

## Overview

This Kubernetes Deployment file deploys **Valkey** in the OpenTelemetry Demo application.

Valkey is an in-memory key-value database that is fully compatible with Redis. It is used for fast data storage and retrieval. In the OpenTelemetry Demo application, Valkey primarily serves as the backend storage for the Cart Service, allowing shopping cart information to be stored and accessed quickly.

For example:

```text
Customer Adds Product
         |
         v
Cart Service
         |
         v
Valkey
         |
         v
Cart Data Stored
```

Since Valkey stores data in memory, operations are extremely fast compared to traditional databases.

---

# Valkey Architecture

```text
Customer
   |
   v
Frontend
   |
   v
Cart Service
   |
   v
Valkey
```

The Cart Service uses Valkey to store and retrieve shopping cart information.

---

# 1. API Version and Resource Type

```yaml
apiVersion: apps/v1
kind: Deployment
```

## Purpose

### apiVersion

Uses the Kubernetes Deployment API.

### kind

Creates a Deployment resource.

### Benefits

* Automatic Pod creation
* Self-healing
* Rolling updates
* Rollback support

---

# 2. Metadata

```yaml
metadata:
  name: opentelemetry-demo-valkey
```

## Purpose

Defines the Deployment name:

```text
opentelemetry-demo-valkey
```

Kubernetes uses this name to manage Valkey Pods.

---

# 3. Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-valkey
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: valkey
  app.kubernetes.io/name: opentelemetry-demo-valkey
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

## Purpose

Labels help Kubernetes:

* Identify resources
* Group components
* Enable monitoring
* Support service discovery
* Select Pods

### Example

```yaml
app.kubernetes.io/component: valkey
```

Identifies this component as the Valkey database.

---

# 4. Deployment Specification

```yaml
spec:
```

Defines how the Valkey Deployment operates.

---

## 4.1 Replicas

```yaml
replicas: 1
```

### Purpose

Runs one Valkey Pod.

```text
Valkey Deployment
        |
        v
    Valkey Pod
```

If the Pod crashes, Kubernetes automatically recreates it.

---

## 4.2 Revision History

```yaml
revisionHistoryLimit: 10
```

### Purpose

Stores the last 10 Deployment versions.

Allows rollback if a new deployment causes issues.

---

# 5. Selector

```yaml
selector:
  matchLabels:
    opentelemetry.io/name: opentelemetry-demo-valkey
```

## Purpose

The Deployment manages Pods with this label.

---

# 6. Pod Template

```yaml
template:
```

Defines how Valkey Pods are created.

---

## Pod Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-valkey
```

Applied automatically to all Valkey Pods.

---

# 7. Service Account

```yaml
serviceAccountName: opentelemetry-demo
```

## Purpose

The Pod runs using the Service Account:

```text
opentelemetry-demo
```

Provides controlled access to Kubernetes resources.

---

# 8. Valkey Container

```yaml
containers:
  - name: valkey
```

This is the main database container.

---

## 8.1 Container Image

```yaml
image: valkey/valkey:7.2-alpine
```

### Breakdown

```text
Registry : Docker Hub
Image    : valkey
Version  : 7.2
Base OS  : Alpine Linux
```

### Purpose

Runs the Valkey in-memory database server.

### Why Alpine?

Alpine Linux is:

* Lightweight
* Secure
* Fast
* Uses less storage

---

## 8.2 Image Pull Policy

```yaml
imagePullPolicy: IfNotPresent
```

### Purpose

* Uses the local image if already present.
* Downloads the image only when necessary.

### Benefit

Reduces startup time and bandwidth usage.

---

# 9. Container Port

```yaml
ports:
  - containerPort: 6379
    name: valkey
```

## Purpose

Valkey listens on:

```text
6379
```

This is the default port used by:

* Redis
* Valkey

### Communication Flow

```text
Cart Service
      |
      v
Valkey:6379
```

---

# 10. OpenTelemetry Configuration

## Service Name

```yaml
OTEL_SERVICE_NAME
```

Obtained dynamically from Pod labels.

Resolved value:

```text
valkey
```

### Purpose

Used for telemetry identification.

---

## Collector Name

```yaml
OTEL_COLLECTOR_NAME=opentelemetry-demo-otelcol
```

### Purpose

Defines the OpenTelemetry Collector service.

---

## Metrics Temporality

```yaml
OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE=cumulative
```

### Purpose

Exports cumulative metrics.

---

# 11. Resource Attributes

```yaml
OTEL_RESOURCE_ATTRIBUTES
```

Resolved value:

```text
service.name=valkey,
service.namespace=opentelemetry-demo,
service.version=1.12.0
```

### Purpose

Adds metadata to telemetry data.

Used by:

* Jaeger
* Grafana
* Prometheus
* OpenTelemetry Collector

---

# 12. Resource Limits

```yaml
resources:
  limits:
    memory: 20Mi
```

## Purpose

Limits memory consumption to:

```text
20 MiB
```

### Why Small?

In the demo environment:

* Only a small amount of cart data is stored.
* No large datasets are maintained.
* Minimal memory is required.

---

# 13. Security Context

```yaml
securityContext:
  runAsGroup: 1000
  runAsNonRoot: true
  runAsUser: 999
```

## Purpose

Runs the container with a non-root user.

### Benefits

* Improved security
* Reduced attack surface
* Follows Kubernetes security best practices

### Configuration

| Setting      | Value | Purpose                 |
| ------------ | ----- | ----------------------- |
| runAsUser    | 999   | Runs as user ID 999     |
| runAsGroup   | 1000  | Runs as group ID 1000   |
| runAsNonRoot | true  | Prevents root execution |

---

# 14. Volume Mounts

```yaml
volumeMounts:
```

No volume mounts are configured.

### Meaning

Valkey data is stored only inside the container.

---

# 15. Volumes

```yaml
volumes:
```

No volumes are defined.

### Meaning

No persistent storage exists.

### Important Note

If the Valkey Pod is deleted:

```text
Stored Cart Data
       |
       v
Lost
```

This is acceptable because this is a demo environment.

In production, a Persistent Volume would normally be used.

---

# Shopping Cart Workflow

```text
Customer Adds Item
        |
        v
Frontend
        |
        v
Cart Service
        |
        v
Valkey
        |
        v
Cart Data Stored
```

---

# Shopping Cart Retrieval

```text
Customer Opens Cart
        |
        v
Cart Service
        |
        v
Valkey
        |
        v
Retrieve Cart Data
        |
        v
Display Cart Contents
```

---

# Observability Flow

```text
Valkey
   |
   v
OpenTelemetry Collector
   |
   +--> Jaeger
   +--> Prometheus
   +--> Grafana
```

Metrics and telemetry related to Valkey can be monitored through the observability stack.

---

# Deployment Startup Flow

```text
Deployment Created
        |
        v
Valkey Pod Created
        |
        v
Container Started
        |
        v
Listening on Port 6379
        |
        v
Ready for Cart Requests
```

---

# Architecture Diagram

```text
Frontend
    |
    v
Cart Service
    |
    v
Valkey Database
    |
    v
Store/Retrieve Cart Data
```

---

# Summary

This Deployment:

* Deploys a Valkey database instance.
* Uses image `valkey/valkey:7.2-alpine`.
* Runs one Pod.
* Listens on port `6379`.
* Stores shopping cart data.
* Uses only `20Mi` of memory.
* Runs as a non-root user.
* Supports self-healing and rolling updates.
* Integrates with the OpenTelemetry observability platform.

## In Simple Terms

Valkey is the **shopping cart database** of the OpenTelemetry Demo application. When customers add products to their cart, the Cart Service stores the cart information in Valkey. Because Valkey keeps data in memory, reading and writing cart data is extremely fast, providing a smooth shopping experience. Kubernetes ensures the Valkey Pod remains running, while OpenTelemetry enables monitoring of its performance and activity.
