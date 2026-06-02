# Recommendation Service (`service.yaml`) Documentation

## Overview

This Kubernetes Service file creates a stable internal network endpoint for the **Recommendation Service** in the OpenTelemetry Demo application.

The Recommendation Service is responsible for generating product recommendations for users. When a customer views a product, this service communicates with the Product 
Catalog Service and returns related or recommended products.

Since Kubernetes Pods can be recreated and their IP addresses can change, this Service provides a permanent DNS name and stable network endpoint that other services can use 
reliably.

---

# Architecture Overview

```text
User
 |
 v
Frontend
 |
 v
Recommendation Service (ClusterIP)
 |
 v
Recommendation Service Pod
 |
 v
Product Recommendations
```

The Service acts as a stable gateway to the Recommendation Service Pod.

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

### Why Service is Needed

Pods are temporary resources.

Example:

```text
Recommendation Pod
IP = 10.0.1.15
```

After restart:

```text
Recommendation Pod
IP = 10.0.1.32
```

Applications using the old IP would fail.

The Service provides a stable endpoint that remains unchanged even when Pods are recreated.

---

# 2. Metadata

```yaml
metadata:
  name: opentelemetry-demo-recommendationservice
```

## Purpose

This is the Service name.

```text
opentelemetry-demo-recommendationservice
```

Kubernetes automatically creates a DNS entry for this Service.

Applications inside the cluster can communicate using:

```text
opentelemetry-demo-recommendationservice:8080
```

instead of Pod IP addresses.

---

# 3. Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-recommendationservice
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: recommendationservice
  app.kubernetes.io/name: opentelemetry-demo-recommendationservice
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

## Purpose

Labels help Kubernetes:

* Organize resources
* Identify components
* Enable monitoring
* Support service discovery
* Select Pods

### Example

```yaml
app.kubernetes.io/component: recommendationservice
```

Indicates that this Service belongs to the Recommendation Service component.

---

# 4. Service Specification

```yaml
spec:
```

Defines how the Service exposes Recommendation Service Pods.

---

# 5. Service Type

```yaml
type: ClusterIP
```

## What is ClusterIP?

ClusterIP exposes the Service only within the Kubernetes cluster.

### Access

```text
Inside Cluster  → Allowed
Outside Cluster → Not Allowed
```

### Why ClusterIP?

The Recommendation Service is an internal backend service.

Only internal microservices need access to it.

Example:

```text
Frontend
    |
    v
Recommendation Service
```

External users never access this service directly.

---

# 6. Port Configuration

```yaml
ports:
  - port: 8080
    name: tcp-service
    targetPort: 8080
```

This configuration exposes the Recommendation Service on port 8080.

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
opentelemetry-demo-recommendationservice:8080
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

Traffic received by the Service is forwarded to port 8080 inside the Recommendation Service container.

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
  opentelemetry.io/name: opentelemetry-demo-recommendationservice
```

## Purpose

The selector identifies which Pods should receive traffic.

Kubernetes searches for Pods containing:

```yaml
opentelemetry.io/name: opentelemetry-demo-recommendationservice
```

---

## Matching Example

### Pod Label

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-recommendationservice
```

### Service Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-recommendationservice
```

Because they match:

```text
Recommendation Service
          |
          v
Recommendation Pod
```

Traffic is routed successfully.

---

# Service Discovery

Kubernetes automatically creates a DNS record.

### DNS Name

```text
opentelemetry-demo-recommendationservice
```

Applications connect using:

```text
opentelemetry-demo-recommendationservice:8080
```

No Pod IP knowledge is required.

---

# How Other Services Use Recommendation Service

The Frontend Service communicates with the Recommendation Service whenever a user views a product.

### Recommendation Flow

```text
User Views Product
        |
        v
Frontend
        |
        v
Recommendation Service
        |
        v
Recommended Products Returned
```

---

# Product Recommendation Workflow

```text
User Selects Product
         |
         v
Recommendation Service
         |
         v
Product Catalog Service
         |
         v
Fetch Product Details
         |
         v
Generate Recommendations
         |
         v
Return Related Products
```

---

# What Happens During Pod Restart?

### Without Service

```text
Recommendation Pod
IP = 10.0.1.15

Pod Restart

New IP = 10.0.1.30

Frontend Cannot Connect
```

### With Service

```text
Recommendation Service
          |
          v
Automatically Routes
To Active Pod
```

Applications continue working without any changes.

---

# Relationship Between Deployment and Service

### Deployment

Creates and manages Recommendation Service Pods.

```text
Recommendation Deployment
           |
           v
Recommendation Pod
```

### Service

Provides stable access to those Pods.

```text
Recommendation Service
          |
          v
Recommendation Pod
```

Together:

```text
Deployment Creates Pods
          |
          v
Service Exposes Pods
```

---

# Complete Recommendation Architecture

```text
User
 |
 v
Frontend
 |
 v
Recommendation Service
 |
 +--> Product Catalog Service
 |
 v
Recommended Products
```

The Recommendation Service depends on the Product Catalog Service to generate recommendations.

---

# Observability Flow

```text
Recommendation Service
          |
          v
OpenTelemetry Collector
          |
          +--> Jaeger
          +--> Prometheus
          +--> Grafana
```

Every recommendation request generates telemetry data.

---

# Architecture Diagram

```text
Frontend
    |
    v
Recommendation Service (ClusterIP)
    |
    v
Recommendation Service Pod
    |
    v
Product Catalog Service
```

---

# Summary

This Service:

* Creates a stable network endpoint for the Recommendation Service.
* Uses the DNS name `opentelemetry-demo-recommendationservice`.
* Exposes port `8080`.
* Routes traffic to container port `8080`.
* Uses `ClusterIP` for internal communication.
* Automatically discovers Recommendation Service Pods using labels.
* Supports Kubernetes DNS-based service discovery.
* Protects applications from Pod IP changes.
* Enables reliable communication between microservices.

## In Simple Terms

The Recommendation Service Service acts as a **permanent internal address** for the Recommendation Service Pod. Instead of communicating directly with a Pod whose IP may change, other services use `opentelemetry-demo-recommendationservice:8080`. Kubernetes automatically forwards requests to the active Recommendation Service Pod, ensuring users always receive product recommendations even if Pods are restarted or replaced.
