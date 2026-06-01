# Cart Service (Service Documentation)

## Overview

This `service.yaml` file creates a Kubernetes **Service** for the Cart Service.

The Cart Service stores and retrieves shopping cart data for users. Since Kubernetes Pods can be recreated at any time and receive new IP addresses, a Service provides a stable endpoint that other microservices can use to communicate with the Cart Service.

This Service exposes the Cart Service internally within the Kubernetes cluster using a **ClusterIP**.

---

# High-Level Service Flow

```text
service.yaml
      ↓
Service Created
      ↓
Cluster IP Assigned
      ↓
Service Finds Cart Service Pods
      ↓
Traffic Routed To Pods
      ↓
Cart Service Accessible Inside Cluster
```

---

# Service Resource

```yaml
apiVersion: v1
kind: Service
```

## Purpose

This tells Kubernetes:

* Use Core API Version 1
* Create a Service resource

### What Happens

When executed:

```bash
kubectl apply -f service.yaml
```

Kubernetes creates a stable networking endpoint for the Cart Service.

---

# Service Name

```yaml
metadata:
  name: opentelemetry-demo-cartservice
```

## Purpose

Provides a unique DNS name for the Cart Service.

### Result

Other services can access Cart Service using:

```text
opentelemetry-demo-cartservice
```

instead of Pod IP addresses.

---

# Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-cartservice
  app.kubernetes.io/component: cartservice
  app.kubernetes.io/name: opentelemetry-demo-cartservice
```

## Purpose

Labels provide metadata that helps Kubernetes organize and identify resources.

### Benefits

* Resource grouping
* Monitoring
* Filtering
* Service discovery
* Troubleshooting

---

# Service Type

```yaml
type: ClusterIP
```

## Purpose

Defines how the Service is exposed.

### What Is ClusterIP?

ClusterIP exposes the application only within the Kubernetes cluster.

### Access Scope

```text
Inside Kubernetes Cluster  → Allowed
Outside Kubernetes Cluster → Not Allowed
```

### Why Used Here?

The Cart Service is consumed by other microservices, not directly by end users.

Example:

```text
Frontend
      ↓
Cart Service
```

---

# Port Configuration

```yaml
ports:
  - port: 8080
    name: tcp-service
    targetPort: 8080
```

This section controls traffic routing.

---

## Service Port

```yaml
port: 8080
```

### Purpose

Defines the port exposed by the Service.

Other applications connect using:

```text
opentelemetry-demo-cartservice:8080
```

---

## Target Port

```yaml
targetPort: 8080
```

### Purpose

Defines which port inside the Pod receives traffic.

### Traffic Flow

```text
Service Port 8080
        ↓
Target Port 8080
        ↓
Cart Service Container
```

---

## Port Name

```yaml
name: tcp-service
```

### Purpose

Provides a readable name for the port.

### Benefits

* Easier troubleshooting
* Easier monitoring
* Better service identification

---

# Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-cartservice
```

## Purpose

Tells the Service which Pods should receive traffic.

### How It Works

The Cart Service Deployment creates Pods with this label:

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-cartservice
```

The Service searches for Pods containing the same label.

### Matching Process

```text
Service Selector
        ↓
Find Matching Pods
        ↓
Attach Pods To Service
        ↓
Route Traffic
```

---

# Relationship Between Deployment and Service

## Deployment Responsibilities

The Deployment:

```text
Creates Pods
Maintains Replica Count
Restarts Failed Pods
Performs Rolling Updates
```

## Service Responsibilities

The Service:

```text
Provides Stable Network Access
Finds Matching Pods
Routes Requests
Load Balances Traffic
```

---

# Why Service Is Required

Without a Service:

```text
Frontend
      ↓
Pod IP (10.244.1.15)
```

Problem:

```text
Pod Restart
      ↓
New Pod Created
      ↓
New IP Assigned
      ↓
Application Communication Breaks
```

---

# How Service Solves This Problem

With Service:

```text
Frontend
      ↓
opentelemetry-demo-cartservice
      ↓
Service Finds Current Pod
      ↓
Request Delivered
```

Even if the Pod changes, the Service name remains the same.

---

# Communication Flow

When a user adds a product to the cart:

```text
Frontend Service
        ↓
Cart Service DNS
(opentelemetry-demo-cartservice)
        ↓
Kubernetes Service
        ↓
Cart Service Pod
        ↓
Port 8080
        ↓
Cart Data Processed
        ↓
Stored In Valkey
        ↓
Response Returned
```

# Complete Runtime Flow

```text
kubectl apply deployment.yaml
            ↓
Cart Service Pod Created
            ↓
Cart Service Starts On Port 8080
            ↓

kubectl apply service.yaml
            ↓
Service Created
            ↓
Cluster IP Assigned
            ↓
Selector Finds Cart Service Pod
            ↓
Stable DNS Name Created
            ↓
Other Services Access:
opentelemetry-demo-cartservice:8080
            ↓
Traffic Routed To Pod
            ↓
Cart Operations Processed
```

---

# Summary

This Service file:

* Creates a stable endpoint for Cart Service
* Exposes Cart Service on port 8080
* Uses ClusterIP networking
* Makes Cart Service accessible inside the cluster
* Automatically discovers Pods using labels
* Routes traffic to Cart Service containers
* Supports load balancing when multiple Pods exist
* Prevents failures caused by changing Pod IP addresses
* Enables reliable communication between microservices

The overall goal of this Service is to provide a stable and reliable way for other services in the OpenTelemetry Demo application to communicate with the Cart Service regardless of Pod restarts, updates, or scaling operations.
