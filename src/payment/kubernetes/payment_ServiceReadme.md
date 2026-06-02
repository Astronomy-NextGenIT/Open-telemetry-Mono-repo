# Payment Service (`service.yaml`) Documentation

## Overview

This Kubernetes Service file creates a stable network endpoint for the **Payment Service** in the OpenTelemetry Demo application.

The Payment Service is responsible for processing customer payments during the checkout process. Since Kubernetes Pods can be recreated and receive new IP addresses, this 
Service provides a permanent DNS name and network endpoint that other microservices can use to communicate with the Payment Service reliably.

Without the Service, applications would need to know the Payment Service Pod's IP address, which can change whenever the Pod is restarted.

---

# Architecture Overview

```text
Customer
    |
    v
Frontend
    |
    v
Checkout Service
    |
    v
Payment Service (ClusterIP Service)
    |
    v
Payment Service Pod
```

The Service acts as a stable entry point for all payment-related requests.

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
Payment Pod
IP = 10.0.1.10
```

After restart:

```text
Payment Pod
IP = 10.0.1.25
```

Applications using the old IP would fail.

A Service provides a permanent address regardless of Pod IP changes.

---

# 2. Metadata

```yaml
metadata:
  name: opentelemetry-demo-paymentservice
```

## Purpose

This is the Service name.

```text
opentelemetry-demo-paymentservice
```

Kubernetes automatically creates a DNS entry for this Service.

Applications inside the cluster can communicate using:

```text
opentelemetry-demo-paymentservice:8080
```

instead of using Pod IP addresses.

---

# 3. Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-paymentservice
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: paymentservice
  app.kubernetes.io/name: opentelemetry-demo-paymentservice
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
app.kubernetes.io/component: paymentservice
```

Indicates that this Service belongs to the Payment Service component.

---

# 4. Service Specification

```yaml
spec:
```

Defines how the Service exposes the Payment Service Pods.

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

The Payment Service is an internal backend service.

Only other microservices need access to it.

Example:

```text
Checkout Service
       |
       v
Payment Service
```

External users never communicate directly with the Payment Service.

---

# 6. Port Configuration

```yaml
ports:
  - port: 8080
    name: tcp-service
    targetPort: 8080
```

This configuration exposes the Payment Service on port 8080.

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
opentelemetry-demo-paymentservice:8080
```

---

## Port Name

```yaml
name: tcp-service
```

### Purpose

Provides a readable name for the port.

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

Traffic received by the Service is forwarded to port 8080 inside the Payment Service container.

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
  opentelemetry.io/name: opentelemetry-demo-paymentservice
```

## Purpose

The selector identifies which Pods should receive traffic.

Kubernetes searches for Pods with the label:

```yaml
opentelemetry.io/name: opentelemetry-demo-paymentservice
```

---

## Matching Example

### Pod Label

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-paymentservice
```

### Service Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-paymentservice
```

Since they match:

```text
Payment Service
       |
       v
Payment Pod
```

Traffic is routed successfully.

---

# Service Discovery

Kubernetes automatically creates a DNS record.

### Service Name

```text
opentelemetry-demo-paymentservice
```

Applications connect using:

```text
opentelemetry-demo-paymentservice:8080
```

No Pod IP knowledge is required.

---

# How Other Services Use Payment Service

The Checkout Service communicates with the Payment Service through this Service.

### Order Processing Flow

```text
Customer Places Order
          |
          v
Checkout Service
          |
          v
opentelemetry-demo-paymentservice:8080
          |
          v
Payment Service Pod
          |
          v
Payment Approved / Rejected
```

The Checkout Service never needs to know the actual Pod IP.

---

# What Happens During Pod Restart?

### Without Service

```text
Payment Pod
IP = 10.0.1.10

Pod Restart

New IP = 10.0.1.20

Checkout Service Fails
```

---

### With Service

```text
Payment Service
       |
       v
Automatically Routes
To Current Payment Pod
```

The application continues working without configuration changes.

---

# Relationship Between Deployment and Service

### Deployment

Creates and manages Payment Service Pods.

```text
Payment Deployment
        |
        v
Payment Pod
```

### Service

Provides stable access to those Pods.

```text
Payment Service
        |
        v
Payment Pod
```

Together:

```text
Deployment Creates Pods
         |
         v
Service Exposes Pods
```

---

# Complete Order Processing Flow

```text
User
 |
 v
Frontend
 |
 v
Checkout Service
 |
 v
Payment Service
 |
 +--> Payment Validation
 |
 +--> Payment Authorization
 |
 v
Payment Result
 |
 v
Shipping Service
 |
 v
Order Completed
```

---

# Service Communication Flow

```text
Checkout Service
        |
        v
opentelemetry-demo-paymentservice:8080
        |
        v
Payment Service
        |
        v
Payment Response
```

---

# Architecture Diagram

```text
Frontend
    |
    v
Checkout Service
    |
    v
Payment Service (ClusterIP)
    |
    v
Payment Service Pod
    |
    v
OpenTelemetry Collector
```

---

# Summary

This Service:

* Creates a stable network endpoint for the Payment Service.
* Uses the DNS name `opentelemetry-demo-paymentservice`.
* Exposes port `8080`.
* Forwards traffic to container port `8080`.
* Uses `ClusterIP` for internal communication.
* Automatically discovers Payment Service Pods using labels.
* Enables reliable service-to-service communication.
* Protects applications from Pod IP changes.
* Supports Kubernetes DNS-based service discovery.

## In Simple Terms

The Payment Service Service acts as a **permanent internal address** for the Payment Service Pod. Instead of connecting directly to a Pod whose IP may change, the Checkout Service communicates with `opentelemetry-demo-paymentservice:8080`. Kubernetes automatically forwards requests to the running Payment Service Pod, ensuring reliable payment processing throughout the application.
