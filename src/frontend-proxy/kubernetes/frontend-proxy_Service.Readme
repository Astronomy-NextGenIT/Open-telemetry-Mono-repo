# Frontend Proxy Service (`service.yaml`) Documentation

## Overview

This Kubernetes Service file creates a stable network endpoint for the **Frontend Proxy** Pod.

In Kubernetes, Pods can be recreated at any time, causing their IP addresses to change. A Service provides a permanent name and IP address that other applications can use to communicate with the Frontend Proxy.

```text
Frontend Proxy Pod
        |
        | (IP may change)
        v
+-------------------------+
| Frontend Proxy Service  |
| Port: 8080             |
+-------------------------+
        |
        v
Other Services / Users
```

---

# 1. API Version and Resource Type

```yaml
apiVersion: v1
kind: Service
```

## Purpose

### apiVersion: v1

Uses the Kubernetes Core API.

### kind: Service

Creates a Kubernetes Service resource.

### Why Service is Needed

Without a Service:

```text
Application --> Pod IP
```

Problem:

```text
Pod Restart
      |
      v
New Pod IP
      |
Application Breaks
```

With a Service:

```text
Application
      |
      v
Service Name
      |
      v
Current Pod
```

Applications always connect using the Service name, regardless of Pod IP changes.

---

# 2. Metadata

```yaml
metadata:
  name: opentelemetry-demo-frontendproxy
```

## Purpose

This is the Service name.

```text
opentelemetry-demo-frontendproxy
```

Other services use this DNS name to communicate with the Frontend Proxy.

Example:

```text
http://opentelemetry-demo-frontendproxy:8080
```

---

# 3. Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-frontendproxy
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: frontendproxy
  app.kubernetes.io/name: opentelemetry-demo-frontendproxy
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

## Purpose of Labels

Labels help organize and identify Kubernetes resources.

### Common Uses

* Monitoring
* Resource grouping
* Filtering resources
* Service discovery

Example:

```yaml
app.kubernetes.io/component: frontendproxy
```

Indicates this Service belongs to the Frontend Proxy component.

---

# 4. Service Specification

```yaml
spec:
```

Defines how the Service exposes the Frontend Proxy Pod.

---

# 5. Service Type

```yaml
type: ClusterIP
```

## What is ClusterIP?

ClusterIP exposes the Service only inside the Kubernetes cluster.

```text
Inside Cluster  --> Allowed
Outside Cluster --> Not Allowed
```

### Access Flow

```text
Frontend Service
       |
       v
Frontend Proxy Service
       |
       v
Frontend Proxy Pod
```

---

## Why ClusterIP is Used

The Frontend Proxy is intended to be accessed internally by other Kubernetes resources.

Benefits:

* Secure
* Internal communication only
* No public exposure
* Lightweight networking

---

# 6. Ports Configuration

```yaml
ports:
  - port: 8080
    name: tcp-service
    targetPort: 8080
```

This defines how traffic reaches the Frontend Proxy Pod.

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

Applications communicate with the Service using this port.

Example:

```text
http://opentelemetry-demo-frontendproxy:8080
```

---

## Port Name

```yaml
name: tcp-service
```

### Purpose

Provides a human-readable name for the port.

Useful when:

* Multiple ports exist
* Monitoring tools inspect services
* Service meshes identify ports

---

## Target Port

```yaml
targetPort: 8080
```

### Purpose

Forwards traffic to Port 8080 inside the Pod.

Flow:

```text
Service Port 8080
        |
        v
Pod Port 8080
```

---

# 7. Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-frontendproxy
```

## Purpose

The selector tells Kubernetes which Pods should receive traffic.

Kubernetes searches for Pods having:

```yaml
opentelemetry.io/name: opentelemetry-demo-frontendproxy
```

---

## Matching Process

### Frontend Proxy Pod

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-frontendproxy
```

### Service Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-frontendproxy
```

Since both match:

```text
Service
   |
   v
Frontend Proxy Pod
```

Traffic is routed successfully.

---

# Traffic Flow

```text
Client Application
        |
        v
opentelemetry-demo-frontendproxy
(Service)
        |
        v
Port 8080
        |
        v
Frontend Proxy Pod
        |
        v
Container Port 8080
```

---

# Service Discovery

Kubernetes automatically creates a DNS entry:

```text
opentelemetry-demo-frontendproxy
```

Other services can access it using:

```text
http://opentelemetry-demo-frontendproxy:8080
```

No need to know Pod IP addresses.

---

# How Service and Deployment Work Together

### Deployment Creates Pods

```text
Deployment
      |
      v
Frontend Proxy Pod
```

### Service Finds Pods Using Labels

```text
Service
      |
      v
Selector Match
      |
      v
Frontend Proxy Pod
```

### Communication Flow

```text
Frontend Service
        |
        v
Frontend Proxy Service
        |
        v
Frontend Proxy Pod
```

---

# What Happens if the Pod Restarts?

### Without Service

```text
Frontend Proxy Pod
IP = 10.1.1.5

Pod Restart

New IP = 10.1.1.9

Applications Fail
```

### With Service

```text
Frontend Proxy Service
        |
        v
Always Points To
Current Running Pod
```

Applications continue working without any changes.

---

# Summary

This Service:

* Creates a stable network endpoint for the Frontend Proxy.
* Uses the name `opentelemetry-demo-frontendproxy`.
* Exposes the application on port `8080`.
* Forwards traffic to container port `8080`.
* Uses the `ClusterIP` type for internal cluster communication.
* Automatically discovers Frontend Proxy Pods using labels.
* Provides DNS-based service discovery.
* Protects applications from Pod IP changes.

## In Simple Terms

This Service acts like a **permanent address for the Frontend Proxy Pod**. Instead of connecting directly to a Pod whose IP may change, other services connect to `opentelemetry-demo-frontendproxy:8080`, and Kubernetes automatically forwards the traffic to the correct running Frontend Proxy Pod.
