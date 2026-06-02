# Shipping Service (`service.yaml`) Documentation

## Overview

This Kubernetes Service file creates a stable internal network endpoint for the **Shipping Service** in the OpenTelemetry Demo application.

The Shipping Service is responsible for calculating shipping costs and delivery information during the checkout process. It communicates with the Quote Service to obtain shipping quotes and returns shipping details to the Checkout Service.

Since Pods are temporary and their IP addresses can change, the Service provides a permanent DNS name and stable endpoint that other microservices can use to communicate with the Shipping Service.

---

# Architecture Overview

```text
Customer
   |
   v
Checkout Service
   |
   v
Shipping Service (ClusterIP)
   |
   v
Shipping Service Pod
   |
   v
Quote Service
```

The Service acts as a stable gateway to the Shipping Service Pod.

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

Pods are dynamic resources.

Example:

```text
Shipping Pod
IP = 10.0.1.20
```

After restart:

```text
Shipping Pod
IP = 10.0.1.45
```

Applications using the old IP would fail.

A Service provides a stable endpoint that remains unchanged even when Pods are recreated.

---

# 2. Metadata

```yaml
metadata:
  name: opentelemetry-demo-shippingservice
```

## Purpose

Defines the Service name:

```text
opentelemetry-demo-shippingservice
```

Kubernetes automatically creates a DNS record for this Service.

Other applications can communicate using:

```text
opentelemetry-demo-shippingservice:8080
```

instead of Pod IP addresses.

---

# 3. Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-shippingservice
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: shippingservice
  app.kubernetes.io/name: opentelemetry-demo-shippingservice
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
app.kubernetes.io/component: shippingservice
```

Identifies this component as the Shipping Service.

---

# 4. Service Specification

```yaml
spec:
```

Defines how the Service exposes Shipping Service Pods.

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

The Shipping Service is an internal backend service.

Only internal microservices need access to it.

Example:

```text
Checkout Service
        |
        v
Shipping Service
```

External users never communicate directly with this service.

---

# 6. Port Configuration

```yaml
ports:
  - port: 8080
    name: tcp-service
    targetPort: 8080
```

This configuration exposes the Shipping Service on port 8080.

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
opentelemetry-demo-shippingservice:8080
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
* Service mesh integrations
* Traffic routing

---

## Target Port

```yaml
targetPort: 8080
```

### Purpose

Traffic received by the Service is forwarded to port 8080 inside the Shipping Service container.

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
  opentelemetry.io/name: opentelemetry-demo-shippingservice
```

## Purpose

The selector identifies which Pods should receive traffic.

Kubernetes searches for Pods containing:

```yaml
opentelemetry.io/name: opentelemetry-demo-shippingservice
```

---

## Matching Example

### Pod Label

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-shippingservice
```

### Service Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-shippingservice
```

Since both labels match:

```text
Shipping Service
       |
       v
Shipping Service Pod
```

Traffic is routed successfully.

---

# Service Discovery

Kubernetes automatically creates a DNS record.

### DNS Name

```text
opentelemetry-demo-shippingservice
```

Applications can connect using:

```text
opentelemetry-demo-shippingservice:8080
```

No Pod IP knowledge is required.

---

# Shipping Request Flow

```text
Customer Checkout
        |
        v
Checkout Service
        |
        v
Shipping Service
        |
        v
Quote Service
        |
        v
Shipping Cost Generated
```

The Shipping Service uses the Quote Service to calculate delivery charges.

---

# What Happens During Pod Restart?

### Without Service

```text
Shipping Pod
IP = 10.0.1.20

Pod Restart

New IP = 10.0.1.45

Applications Fail
```

### With Service

```text
Shipping Service
       |
       v
Automatically Routes
To Active Pod
```

Applications continue working normally.

---

# Relationship Between Deployment and Service

### Deployment

Creates and manages Shipping Service Pods.

```text
Shipping Deployment
         |
         v
Shipping Pod
```

### Service

Provides stable access to those Pods.

```text
Shipping Service
        |
        v
Shipping Pod
```

Together:

```text
Deployment Creates Pods
          |
          v
Service Exposes Pods
```

---

# Complete Shipping Workflow

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
Shipping Service
   |
   v
Quote Service
   |
   v
Shipping Charges Returned
```

---

# Observability Flow

```text
Shipping Service
       |
       v
OpenTelemetry Collector
       |
       +--> Jaeger
       +--> Prometheus
       +--> Grafana
```

All shipping-related requests generate telemetry data.

---

# Architecture Diagram

```text
Checkout Service
       |
       v
Shipping Service (ClusterIP)
       |
       v
Shipping Service Pod
       |
       v
Quote Service
```

---

# Summary

This Service:

* Creates a stable network endpoint for the Shipping Service.
* Uses the DNS name `opentelemetry-demo-shippingservice`.
* Exposes port `8080`.
* Routes traffic to container port `8080`.
* Uses `ClusterIP` for internal communication.
* Automatically discovers Shipping Service Pods using labels.
* Supports Kubernetes DNS-based service discovery.
* Protects applications from Pod IP changes.
* Enables reliable communication between microservices.

## In Simple Terms

The Shipping Service Service acts as a **permanent internal address** for the Shipping Service Pod. Instead of connecting directly to a Pod whose IP can change, other services communicate with `opentelemetry-demo-shippingservice:8080`. Kubernetes automatically forwards requests to the active Shipping Service Pod, ensuring shipping cost calculations and delivery information remain available even when Pods are restarted or replaced.
