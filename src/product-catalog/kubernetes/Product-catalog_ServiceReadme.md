# Product Catalog Service (`service.yaml`) Documentation

## Overview

This Kubernetes Service file creates a stable network endpoint for the **Product Catalog Service** in the OpenTelemetry Demo application.

The Product Catalog Service provides product information such as product names, descriptions, prices, categories, and images. Other microservices, especially the Frontend Service, use this Service to retrieve product data.

Since Kubernetes Pods can restart and receive new IP addresses, the Service provides a permanent DNS name and network endpoint that applications can use reliably.

---

# Architecture Overview

```text
User
 |
 v
Frontend
 |
 v
Product Catalog Service (ClusterIP)
 |
 v
Product Catalog Pod
 |
 v
Product Information
```

The Service acts as a stable gateway to the Product Catalog Pod.

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
Product Catalog Pod
IP = 10.0.1.10
```

After restart:

```text
Product Catalog Pod
IP = 10.0.1.25
```

Applications using the old IP would fail.

The Service solves this problem by providing a permanent network endpoint.

---

# 2. Metadata

```yaml
metadata:
  name: opentelemetry-demo-productcatalogservice
```

## Purpose

This is the Service name.

```text
opentelemetry-demo-productcatalogservice
```

Kubernetes automatically creates a DNS record for this Service.

Applications can access it using:

```text
opentelemetry-demo-productcatalogservice:8080
```

instead of using Pod IP addresses.

---

# 3. Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-productcatalogservice
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: productcatalogservice
  app.kubernetes.io/name: opentelemetry-demo-productcatalogservice
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

## Purpose

Labels help Kubernetes:

* Organize resources
* Identify components
* Select Pods
* Enable monitoring
* Support service discovery

### Example

```yaml
app.kubernetes.io/component: productcatalogservice
```

Shows that this Service belongs to the Product Catalog Service component.

---

# 4. Service Specification

```yaml
spec:
```

Defines how the Service exposes the Product Catalog Pods.

---

# 5. Service Type

```yaml
type: ClusterIP
```

## What is ClusterIP?

ClusterIP exposes the Service only inside the Kubernetes cluster.

### Access

```text
Inside Cluster  → Allowed
Outside Cluster → Not Allowed
```

### Why ClusterIP?

The Product Catalog Service is an internal backend service.

Only internal microservices need access.

Example:

```text
Frontend Service
       |
       v
Product Catalog Service
```

External users do not access this service directly.

---

# 6. Port Configuration

```yaml
ports:
  - port: 8080
    name: tcp-service
    targetPort: 8080
```

This configuration exposes the Product Catalog Service on port 8080.

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
opentelemetry-demo-productcatalogservice:8080
```

---

## Port Name

```yaml
name: tcp-service
```

### Purpose

Provides a readable name for the port.

Useful for:

* Monitoring tools
* Service meshes
* Traffic management

---

## Target Port

```yaml
targetPort: 8080
```

### Purpose

Traffic received by the Service is forwarded to port 8080 inside the Product Catalog container.

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
  opentelemetry.io/name: opentelemetry-demo-productcatalogservice
```

## Purpose

The selector identifies which Pods should receive traffic.

Kubernetes searches for Pods containing:

```yaml
opentelemetry.io/name: opentelemetry-demo-productcatalogservice
```

---

## Matching Example

### Pod Label

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-productcatalogservice
```

### Service Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-productcatalogservice
```

Because they match:

```text
Product Catalog Service
          |
          v
Product Catalog Pod
```

Traffic is routed successfully.

---

# Service Discovery

Kubernetes automatically creates a DNS record.

### Service Name

```text
opentelemetry-demo-productcatalogservice
```

Applications connect using:

```text
opentelemetry-demo-productcatalogservice:8080
```

No Pod IP knowledge is required.

---

# How Other Services Use Product Catalog Service

The Frontend Service communicates with the Product Catalog Service whenever users browse products.

### Product Browsing Flow

```text
User Opens Website
          |
          v
Frontend Service
          |
          v
Product Catalog Service
          |
          v
Product Data Returned
          |
          v
Products Displayed
```

---

# Product Search Flow

```text
User Searches Product
          |
          v
Frontend
          |
          v
Product Catalog Service
          |
          +--> Product Name
          +--> Description
          +--> Price
          +--> Image
          |
          v
Search Results Displayed
```

---

# What Happens During Pod Restart?

### Without Service

```text
Product Catalog Pod
IP = 10.0.1.10

Pod Restart

New IP = 10.0.1.20

Frontend Fails
```

### With Service

```text
Product Catalog Service
          |
          v
Automatically Routes
To Current Running Pod
```

Applications continue working without configuration changes.

---

# Relationship Between Deployment and Service

### Deployment

Creates and manages Product Catalog Pods.

```text
Product Catalog Deployment
           |
           v
Product Catalog Pod
```

### Service

Provides stable access to those Pods.

```text
Product Catalog Service
           |
           v
Product Catalog Pod
```

Together:

```text
Deployment Creates Pods
           |
           v
Service Exposes Pods
```

---

# Complete Store Flow

```text
User
 |
 v
Frontend
 |
 +--> Product Catalog Service
 |          |
 |          v
 |      Product Data
 |
 +--> Cart Service
 |
 +--> Checkout Service
 |
 v
Order Completed
```

The Product Catalog Service is one of the first services involved when users interact with the store.

---

# Observability Flow

```text
Product Catalog Service
          |
          v
OpenTelemetry Collector
          |
          +--> Jaeger
          +--> Prometheus
          +--> Grafana
```

Every product request generates telemetry data.

---

# Architecture Diagram

```text
Frontend
    |
    v
Product Catalog Service (ClusterIP)
    |
    v
Product Catalog Pod
    |
    v
Product Database/Data Source
```

---

# Summary

This Service:

* Creates a stable network endpoint for the Product Catalog Service.
* Uses the DNS name `opentelemetry-demo-productcatalogservice`.
* Exposes port `8080`.
* Forwards traffic to container port `8080`.
* Uses `ClusterIP` for internal communication.
* Automatically discovers Product Catalog Pods using labels.
* Supports Kubernetes DNS-based service discovery.
* Protects applications from Pod IP changes.
* Enables reliable product data access across microservices.

## In Simple Terms

The Product Catalog Service acts as a **permanent internal address** for the Product Catalog Pod. Whenever the Frontend needs product information, it connects to `opentelemetry-demo-productcatalogservice:8080` instead of directly contacting a Pod. Kubernetes automatically routes the request to the correct running Pod, ensuring users can always browse and view products even if Pods are restarted or replaced.
