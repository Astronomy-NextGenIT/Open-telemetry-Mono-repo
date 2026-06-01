# Quote Service (`service.yaml`) Documentation

## Overview

This Kubernetes Service file creates a stable internal network endpoint for the **Quote Service** in the OpenTelemetry Demo application.

The Quote Service processes quote-related requests and provides quote data to other microservices. Since Kubernetes Pods are temporary and their IP addresses can change when restarted, the Service provides a permanent DNS name and network endpoint that other services can use reliably.

The Service ensures uninterrupted communication between microservices even if the Quote Service Pod is recreated.

---

# Architecture Overview

```text
Application
     |
     v
Quote Service (ClusterIP)
     |
     v
Quote Service Pod
```

The Service acts as a stable gateway to the Quote Service Pod.

---

# 1. API Version and Resource Type

```yaml
apiVersion: v1
kind: Service
```

## Purpose

### apiVersion

```yaml
v1
```

Uses the Kubernetes Core API.

### kind

```yaml
Service
```

Creates a Kubernetes Service resource.

### Why Service is Required

Pods are temporary resources.

Example:

```text
Quote Service Pod
IP = 10.0.1.15
```

After a restart:

```text
Quote Service Pod
IP = 10.0.1.30
```

Applications using the old IP would fail.

A Service provides a permanent endpoint that remains unchanged regardless of Pod restarts.

---

# 2. Metadata

```yaml
metadata:
  name: opentelemetry-demo-quoteservice
```

## Purpose

Defines the Service name:

```text
opentelemetry-demo-quoteservice
```

Kubernetes automatically creates a DNS record for this Service.

Applications can communicate using:

```text
opentelemetry-demo-quoteservice:8080
```

instead of using Pod IP addresses.

---

# 3. Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-quoteservice
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: quoteservice
  app.kubernetes.io/name: opentelemetry-demo-quoteservice
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

## Purpose

Labels help Kubernetes:

* Identify resources
* Group related components
* Enable monitoring
* Support service discovery
* Select matching Pods

### Example

```yaml
app.kubernetes.io/component: quoteservice
```

Indicates that this Service belongs to the Quote Service component.

---

# 4. Service Specification

```yaml
spec:
```

Defines how the Service exposes Quote Service Pods.

---

# 5. Service Type

```yaml
type: ClusterIP
```

## What is ClusterIP?

ClusterIP exposes the Service only within the Kubernetes cluster.

### Accessibility

```text
Inside Cluster  → Allowed
Outside Cluster → Not Allowed
```

### Why ClusterIP?

The Quote Service is an internal backend service.

Only other microservices need access to it.

Example:

```text
Frontend
    |
    v
Quote Service
```

External users never communicate directly with the Quote Service.

---

# 6. Port Configuration

```yaml
ports:
  - port: 8080
    name: tcp-service
    targetPort: 8080
```

This configuration exposes the Quote Service on port 8080.

---

## Service Port

```yaml
port: 8080
```

### Purpose

The Service listens on:

```text
8080
```

Applications communicate using:

```text
opentelemetry-demo-quoteservice:8080
```

---

## Port Name

```yaml
name: tcp-service
```

### Purpose

Provides a human-readable name for the port.

Useful for:

* Monitoring tools
* Service meshes
* Traffic routing

---

## Target Port

```yaml
targetPort: 8080
```

### Purpose

Traffic received on the Service port is forwarded to port 8080 inside the Quote Service container.

Flow:

```text
Service Port 8080
       |
       v
Container Port 8080
```

---

# 7. Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-quoteservice
```

## Purpose

The selector identifies which Pods should receive traffic.

Kubernetes searches for Pods containing:

```yaml
opentelemetry.io/name: opentelemetry-demo-quoteservice
```

---

## Matching Example

### Pod Label

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-quoteservice
```

### Service Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-quoteservice
```

Since both labels match:

```text
Quote Service
      |
      v
Quote Service Pod
```

Traffic is routed successfully.

---

# Service Discovery

Kubernetes automatically creates a DNS record.

### DNS Name

```text
opentelemetry-demo-quoteservice
```

Applications can connect using:

```text
opentelemetry-demo-quoteservice:8080
```

No Pod IP knowledge is required.

---

# Communication Flow

```text
Application
     |
     v
opentelemetry-demo-quoteservice:8080
     |
     v
Quote Service Pod
     |
     v
Quote Response
```

The Service ensures reliable communication with the running Pod.

---

# What Happens During Pod Restart?

### Without Service

```text
Quote Service Pod
IP = 10.0.1.15

Pod Restart

New IP = 10.0.1.30

Applications Fail
```

---

### With Service

```text
Quote Service
      |
      v
Automatically Routes
To Current Running Pod
```

Applications continue working normally.

---

# Relationship Between Deployment and Service

### Deployment

Creates and manages Quote Service Pods.

```text
Quote Service Deployment
           |
           v
     Quote Service Pod
```

### Service

Provides stable access to those Pods.

```text
Quote Service Service
          |
          v
     Quote Service Pod
```

Together:

```text
Deployment Creates Pods
          |
          v
Service Exposes Pods
```

---

# Observability Flow

```text
Quote Service
      |
      v
OpenTelemetry Collector
      |
      +--> Jaeger
      +--> Prometheus
      +--> Grafana
```

All quote-related requests can be monitored through observability tools.

---

# Architecture Diagram

```text
Frontend
    |
    v
Quote Service (ClusterIP)
    |
    v
Quote Service Pod
    |
    v
Quote Processing Logic
```

---

# Summary

This Service:

* Creates a stable network endpoint for the Quote Service.
* Uses the DNS name `opentelemetry-demo-quoteservice`.
* Exposes port `8080`.
* Routes traffic to container port `8080`.
* Uses `ClusterIP` for internal communication.
* Automatically discovers Quote Service Pods using labels.
* Supports Kubernetes DNS-based service discovery.
* Protects applications from Pod IP changes.
* Enables reliable microservice communication.

## In Simple Terms

The Quote Service Service acts as a **permanent internal address** for the Quote Service Pod. Instead of connecting directly to a Pod whose IP can change, other services communicate with `opentelemetry-demo-quoteservice:8080`. Kubernetes automatically forwards requests to the active Quote Service Pod, ensuring reliable communication and uninterrupted application functionality.
