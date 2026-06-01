# Load Generator Service (`service.yaml`) Documentation

## Overview

This Kubernetes Service file creates a stable network endpoint for the **Load Generator Service** in the OpenTelemetry Demo application.

The Load Generator uses **Locust** to simulate user traffic against the application. This Service provides a permanent DNS name and network endpoint so that other 
components (such as Frontend Proxy) can access the Locust Web UI without needing to know the Pod's IP address.

Since Pod IP addresses can change when Pods restart, Kubernetes Services provide stable connectivity.

---

# Architecture Overview

```text
Kubernetes Cluster
        |
        v
Load Generator Service
(opentelemetry-demo-loadgenerator)
        |
        v
Load Generator Pod
(Locust)
        |
        v
Frontend Proxy
        |
        v
Frontend Application
```

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

Creates a Kubernetes Service resource.

### Why Service is Required

Pods are temporary resources.

Example:

```text
Load Generator Pod
IP = 10.10.1.10
```

After restart:

```text
Load Generator Pod
IP = 10.10.1.25
```

Without a Service, applications would need to update their configuration every time the Pod IP changes.

The Service solves this by providing a permanent endpoint.

---

# 2. Metadata

```yaml
metadata:
  name: opentelemetry-demo-loadgenerator
```

## Purpose

This is the Service name.

```text
opentelemetry-demo-loadgenerator
```

Kubernetes automatically creates a DNS entry.

Applications inside the cluster can access the service using:

```text
http://opentelemetry-demo-loadgenerator:8089
```

---

# 3. Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-loadgenerator
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: loadgenerator
  app.kubernetes.io/name: opentelemetry-demo-loadgenerator
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

## Purpose

Labels provide metadata for Kubernetes resources.

### Uses

* Resource identification
* Resource grouping
* Monitoring
* Pod selection
* Service discovery

Example:

```yaml
app.kubernetes.io/component: loadgenerator
```

Indicates that this Service belongs to the Load Generator component.

---

# 4. Service Specification

```yaml
spec:
```

Defines how the Service exposes the Load Generator Pods.

---

# 5. Service Type

```yaml
type: ClusterIP
```

## What is ClusterIP?

ClusterIP exposes the Service only inside the Kubernetes cluster.

### Accessibility

```text
Inside Cluster  → Accessible
Outside Cluster → Not Accessible
```

### Why ClusterIP?

The Load Generator is an internal utility service.

It is not intended for public internet access.

Instead:

```text
Developers
      |
      v
Frontend Proxy
      |
      v
Load Generator Service
```

The service is used only within the cluster.

---

# 6. Port Configuration

```yaml
ports:
  - port: 8089
    name: tcp-service
    targetPort: 8089
```

This configuration exposes the Locust Web UI.

---

## Service Port

```yaml
port: 8089
```

### Purpose

The Service listens on:

```text
8089
```

Applications can connect using:

```text
http://opentelemetry-demo-loadgenerator:8089
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
* Multi-port services

---

## Target Port

```yaml
targetPort: 8089
```

### Purpose

Traffic received by the Service is forwarded to port 8089 inside the Load Generator container.

Flow:

```text
Service Port 8089
        |
        v
Container Port 8089
```

---

# 7. Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-loadgenerator
```

## Purpose

The selector identifies which Pods should receive traffic.

Kubernetes searches for Pods with:

```yaml
opentelemetry.io/name: opentelemetry-demo-loadgenerator
```

---

## Matching Example

### Pod Label

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-loadgenerator
```

### Service Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-loadgenerator
```

Since the labels match:

```text
Load Generator Service
          |
          v
Load Generator Pod
```

Traffic is routed successfully.

---

# Service Discovery

Kubernetes automatically creates a DNS record.

### Service Name

```text
opentelemetry-demo-loadgenerator
```

Applications can connect using:

```text
http://opentelemetry-demo-loadgenerator:8089
```

No Pod IP knowledge is required.

---

# Locust Web UI Access

The Load Generator runs Locust, which provides a web interface.

This Service exposes that interface.

### Locust UI

```text
http://opentelemetry-demo-loadgenerator:8089
```

Administrators can view:

* Active users
* Requests per second
* Response times
* Error rates
* Traffic statistics

---

# Relationship Between Deployment and Service

### Deployment

Responsible for creating and managing Pods.

```text
Load Generator Deployment
            |
            v
Load Generator Pod
```

### Service

Responsible for providing stable access.

```text
Load Generator Service
            |
            v
Load Generator Pod
```

Together:

```text
Deployment Creates Pods
            |
            v
Service Exposes Pods
```

---

# What Happens During Pod Restart?

### Without Service

```text
Load Generator Pod
IP = 10.0.1.5

Pod Restart

New IP = 10.0.1.20

Connections Fail
```

### With Service

```text
Load Generator Service
           |
           v
Always Routes To
Current Running Pod
```

Applications continue working normally.

---

# Load Testing Traffic Flow

```text
Load Generator Pod
        |
        v
Frontend Proxy
        |
        v
Frontend Service
        |
        +--> Product Catalog Service
        +--> Cart Service
        +--> Checkout Service
        +--> Recommendation Service
        +--> Shipping Service
```

The generated traffic creates telemetry data for the entire application.

---

# Observability Flow

```text
Load Generator
       |
       v
Frontend Proxy
       |
       v
Microservices
       |
       v
OpenTelemetry Collector
       |
       +--> Jaeger
       +--> Prometheus
       +--> Grafana
```

This traffic allows observability tools to display meaningful data.

---

# Architecture Diagram

```text
Load Generator Service
(opentelemetry-demo-loadgenerator)
                |
                v
         Load Generator Pod
                |
                v
        Frontend Proxy
                |
                v
            Frontend
                |
                +--> Cart Service
                +--> Checkout Service
                +--> Product Catalog Service
                +--> Recommendation Service
                +--> Shipping Service
```

---

# Summary

This Service:

* Creates a stable network endpoint for the Load Generator.
* Uses the DNS name `opentelemetry-demo-loadgenerator`.
* Exposes port `8089`.
* Forwards traffic to container port `8089`.
* Uses `ClusterIP` for internal communication.
* Automatically discovers Load Generator Pods using labels.
* Exposes the Locust Web UI.
* Provides DNS-based service discovery.
* Protects applications from Pod IP changes.

## In Simple Terms

The Load Generator Service acts as a **permanent internal address** for the Locust-based Load Generator. Instead of connecting directly to a Pod whose IP may change, Kubernetes components use `opentelemetry-demo-loadgenerator:8089`. The Service automatically forwards requests to the running Load Generator Pod, ensuring reliable access to the Locust UI and traffic generation functionality.
