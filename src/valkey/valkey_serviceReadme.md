# Valkey Service (`service.yaml`) Documentation

## Overview

This Kubernetes Service file creates a stable internal network endpoint for **Valkey** in the OpenTelemetry Demo application.

Valkey is an in-memory key-value database (Redis-compatible) used by the application to store and retrieve shopping cart data quickly. Since Kubernetes Pods can be restarted 
and receive new IP addresses, the Service provides a permanent DNS name and stable endpoint that applications can use to access Valkey reliably.

Without this Service, applications would need to know the changing IP address of the Valkey Pod.

---

# Architecture Overview

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
Valkey Service (ClusterIP)
   |
   v
Valkey Pod
```

The Service acts as a stable gateway to the Valkey database Pod.

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
Valkey Pod
IP = 10.0.1.50
```

After restart:

```text
Valkey Pod
IP = 10.0.1.78
```

Applications using the old IP would fail.

The Service provides a stable endpoint that remains unchanged even when Pods are recreated.

---

# 2. Metadata

```yaml
metadata:
  name: opentelemetry-demo-valkey
```

## Purpose

Defines the Service name:

```text
opentelemetry-demo-valkey
```

Kubernetes automatically creates a DNS record for this Service.

Applications can connect using:

```text
opentelemetry-demo-valkey:6379
```

instead of using Pod IP addresses.

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
* Organize components
* Support service discovery
* Enable monitoring
* Select Pods

### Example

```yaml
app.kubernetes.io/component: valkey
```

Indicates that this resource belongs to the Valkey database component.

---

# 4. Service Specification

```yaml
spec:
```

Defines how the Service exposes Valkey Pods.

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

Valkey is an internal database.

Only application services need access to it.

Example:

```text
Cart Service
     |
     v
Valkey
```

External users should never access the database directly.

---

# 6. Port Configuration

```yaml
ports:
  - port: 6379
    name: valkey
    targetPort: 6379
```

This configuration exposes the Valkey database on port 6379.

---

## Service Port

```yaml
port: 6379
```

### Purpose

The Service listens on:

```text
6379
```

Applications communicate using:

```text
opentelemetry-demo-valkey:6379
```

---

## Port Name

```yaml
name: valkey
```

### Purpose

Provides a descriptive name for the port.

Useful for:

* Monitoring
* Service meshes
* Traffic routing
* Kubernetes networking

---

## Target Port

```yaml
targetPort: 6379
```

### Purpose

Traffic received on the Service port is forwarded to port 6379 inside the Valkey container.

Flow:

```text
Service Port 6379
        |
        v
Container Port 6379
```

---

# 7. Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-valkey
```

## Purpose

The selector determines which Pods receive traffic.

Kubernetes searches for Pods containing:

```yaml
opentelemetry.io/name: opentelemetry-demo-valkey
```

---

## Matching Example

### Pod Label

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-valkey
```

### Service Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-valkey
```

Since the labels match:

```text
Valkey Service
      |
      v
Valkey Pod
```

Traffic is routed successfully.

---

# Service Discovery

Kubernetes automatically creates a DNS record.

### DNS Name

```text
opentelemetry-demo-valkey
```

Applications connect using:

```text
opentelemetry-demo-valkey:6379
```

No Pod IP address is required.

---

# Cart Service Integration

The Cart Service uses Valkey to store shopping cart information.

### Data Storage Flow

```text
Customer Adds Product
         |
         v
Cart Service
         |
         v
Valkey Service
         |
         v
Valkey Pod
         |
         v
Cart Data Stored
```

---

### Data Retrieval Flow

```text
Customer Opens Cart
         |
         v
Cart Service
         |
         v
Valkey Service
         |
         v
Valkey Pod
         |
         v
Cart Data Retrieved
```

---

# What Happens During Pod Restart?

### Without Service

```text
Valkey Pod
IP = 10.0.1.50

Pod Restart

New IP = 10.0.1.78

Cart Service Fails
```

### With Service

```text
Valkey Service
      |
      v
Automatically Routes
To Active Pod
```

Applications continue working normally.

---

# Relationship Between Deployment and Service

### Deployment

Creates and manages Valkey Pods.

```text
Valkey Deployment
        |
        v
Valkey Pod
```

### Service

Provides stable access to those Pods.

```text
Valkey Service
      |
      v
Valkey Pod
```

Together:

```text
Deployment Creates Pods
          |
          v
Service Exposes Pods
```

---

# Complete Cart Architecture

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
Valkey Service
   |
   v
Valkey Pod
```

The Service ensures the Cart Service can always locate the Valkey database.

---

# Observability Perspective

Although the Service itself does not generate application telemetry, it enables communication between application components whose activity is monitored through OpenTelemetry.

```text
Cart Service
      |
      v
Valkey
      |
      v
OpenTelemetry Ecosystem
```

---

# Architecture Diagram

```text
Cart Service
      |
      v
Valkey Service (ClusterIP)
      |
      v
Valkey Pod
      |
      v
Shopping Cart Data
```

---

# Summary

This Service:

* Creates a stable network endpoint for Valkey.
* Uses the DNS name `opentelemetry-demo-valkey`.
* Exposes port `6379`.
* Routes traffic to container port `6379`.
* Uses `ClusterIP` for internal communication.
* Automatically discovers Valkey Pods using labels.
* Supports Kubernetes DNS-based service discovery.
* Protects applications from Pod IP changes.
* Enables reliable database connectivity for the Cart Service.

## In Simple Terms

The Valkey Service acts as a **permanent internal address** for the Valkey database Pod. Instead of connecting directly to a Pod whose IP address may change, applications use `opentelemetry-demo-valkey:6379`. Kubernetes automatically forwards requests to the active Valkey Pod, ensuring that shopping cart data can always be stored and retrieved reliably, even if the database Pod is restarted or replaced.
