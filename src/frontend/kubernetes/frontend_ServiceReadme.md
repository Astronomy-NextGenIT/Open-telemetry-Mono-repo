# Frontend Service (`service.yaml`) Documentation

## Overview

This Kubernetes Service file creates a stable network endpoint for the **Frontend Service**.

The Frontend Pod hosts the main web application of the OpenTelemetry Demo. Since Pod IP addresses can change when Pods restart, Kubernetes uses a Service to provide a permanent DNS name and IP address for accessing the Frontend.

```text
Frontend Pod
      |
      | (IP may change)
      v
+----------------------+
| Frontend Service     |
| Port: 8080          |
+----------------------+
      |
      v
Frontend Proxy
```

Without this Service, other applications would need to know the Pod's IP address, which is unreliable.

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

Uses Kubernetes Core API resources.

### kind

```yaml
Service
```

Creates a Kubernetes Service.

### Why Service is Required

Pods are temporary resources.

Example:

```text
Frontend Pod
IP = 10.10.1.5
```

After restart:

```text
Frontend Pod
IP = 10.10.1.20
```

Applications depending on the old IP would fail.

The Service solves this problem by providing a permanent endpoint.

---

# 2. Metadata

```yaml
metadata:
  name: opentelemetry-demo-frontend
```

## Purpose

This is the Service name.

```text
opentelemetry-demo-frontend
```

Kubernetes automatically creates DNS records for this Service.

Other applications can access it using:

```text
http://opentelemetry-demo-frontend:8080
```

---

# 3. Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-frontend
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: frontend
  app.kubernetes.io/name: opentelemetry-demo-frontend
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

## Purpose

Labels are metadata attached to Kubernetes resources.

### Uses

* Resource organization
* Monitoring
* Service discovery
* Pod selection

Example:

```yaml
app.kubernetes.io/component: frontend
```

Indicates that this Service belongs to the Frontend component.

---

# 4. Service Specification

```yaml
spec:
```

Defines how the Service exposes the Frontend Pods.

---

# 5. Service Type

```yaml
type: ClusterIP
```

## What is ClusterIP?

ClusterIP exposes the Service only inside the Kubernetes cluster.

### Accessibility

```text
Inside Cluster  -> Allowed
Outside Cluster -> Not Allowed
```

### Communication Flow

```text
Frontend Proxy
      |
      v
Frontend Service
      |
      v
Frontend Pod
```

---

## Why ClusterIP?

The Frontend Service does not need direct internet access.

External users access the application through:

```text
Internet
    |
    v
ALB Ingress
    |
    v
Frontend Proxy
    |
    v
Frontend Service
```

This architecture improves security by keeping backend services private.

---

# 6. Ports Configuration

```yaml
ports:
  - port: 8080
    name: tcp-service
    targetPort: 8080
```

Defines how traffic is forwarded to the Frontend Pod.

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

Applications connect using:

```text
http://opentelemetry-demo-frontend:8080
```

---

## Port Name

```yaml
name: tcp-service
```

### Purpose

Provides a human-readable name for the port.

Useful for:

* Monitoring systems
* Service meshes
* Multi-port services

---

## Target Port

```yaml
targetPort: 8080
```

### Purpose

Traffic received by the Service is forwarded to port 8080 inside the Frontend container.

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
  opentelemetry.io/name: opentelemetry-demo-frontend
```

## Purpose

The selector identifies which Pods should receive traffic.

Kubernetes searches for Pods with the label:

```yaml
opentelemetry.io/name: opentelemetry-demo-frontend
```

---

## Matching Example

### Frontend Pod Label

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-frontend
```

### Service Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-frontend
```

Because both match:

```text
Frontend Service
        |
        v
Frontend Pod
```

Traffic is routed successfully.

---

# Service Discovery

Kubernetes automatically creates a DNS record.

### Service Name

```text
opentelemetry-demo-frontend
```

Applications can access the Frontend using:

```text
http://opentelemetry-demo-frontend:8080
```

No need to know Pod IP addresses.

---

# How Frontend Proxy Uses This Service

In the Frontend Proxy Deployment:

```yaml
FRONTEND_HOST=opentelemetry-demo-frontend
FRONTEND_PORT=8080
```

The Frontend Proxy connects to:

```text
http://opentelemetry-demo-frontend:8080
```

The Service then forwards requests to the actual Frontend Pod.

---

# Request Flow

```text
User Browser
      |
      v
AWS ALB
      |
      v
Frontend Proxy Ingress
      |
      v
Frontend Proxy Service
      |
      v
Frontend Proxy Pod
      |
      v
Frontend Service
      |
      v
Frontend Pod
```

---

# What Happens During Pod Restart?

### Without Service

```text
Frontend Pod
IP = 10.0.1.5

Pod Restart

New IP = 10.0.1.8

Connections Fail
```

---

### With Service

```text
Frontend Service
        |
        v
Always Routes To
Current Running Pod
```

Applications continue working even if Pod IPs change.

---

# Relationship Between Deployment and Service

### Deployment

Responsible for creating and managing Pods.

```text
Frontend Deployment
        |
        v
Frontend Pod
```

### Service

Responsible for providing stable access to Pods.

```text
Frontend Service
        |
        v
Frontend Pod
```

Together:

```text
Deployment Creates Pods
        |
        v
Service Exposes Pods
```

---

# Architecture Diagram

```text
Internet User
      |
      v
AWS ALB
      |
      v
Frontend Proxy Ingress
      |
      v
Frontend Proxy Service
      |
      v
Frontend Proxy Pod
      |
      v
Frontend Service
      |
      v
Frontend Pod
      |
      +--> Product Catalog Service
      +--> Cart Service
      +--> Checkout Service
      +--> Recommendation Service
      +--> Currency Service
      +--> Shipping Service
      +--> Ad Service
```

---

# Summary

This Service:

* Creates a stable network endpoint for the Frontend application.
* Uses the name `opentelemetry-demo-frontend`.
* Exposes port `8080`.
* Forwards traffic to container port `8080`.
* Uses `ClusterIP` for internal cluster communication.
* Automatically discovers Frontend Pods using labels.
* Provides DNS-based service discovery.
* Protects applications from Pod IP changes.

## In Simple Terms

The Frontend Service acts like a **permanent internal address** for the Frontend application. Instead of connecting directly to a Frontend Pod whose IP can change, other services such as the Frontend Proxy connect to `opentelemetry-demo-frontend:8080`, and Kubernetes automatically forwards the request to the currently running Frontend Pod. This ensures reliable communication within the cluster.
