# Checkout Service Service Documentation

## Overview

This `service.yaml` file creates a Kubernetes **Service** for the Checkout Service.

The Checkout Service is responsible for coordinating the complete order placement workflow. Since Kubernetes Pods can be recreated at any time and receive new IP addresses, this Service provides a stable network endpoint that other services can use to communicate with the Checkout Service.

The Service ensures reliable communication between the Frontend and the Checkout Service regardless of Pod restarts, updates, or scaling operations.

---

# High-Level Service Flow

```text
service.yaml
      ↓
Service Created
      ↓
Cluster IP Assigned
      ↓
Service Finds Checkout Pods
      ↓
Traffic Routed To Pods
      ↓
Checkout Service Accessible Inside Cluster
```

---

# Service Resource

```yaml
apiVersion: v1
kind: Service
```

## Purpose

Defines a Kubernetes Service resource.

### What Happens

When executed:

```bash
kubectl apply -f service.yaml
```

Kubernetes creates a stable networking endpoint for the Checkout Service.

---

# Service Name

```yaml
metadata:
  name: opentelemetry-demo-checkoutservice
```

## Purpose

Provides a DNS name for the Checkout Service.

### Result

Other applications can access the Checkout Service using:

```text
opentelemetry-demo-checkoutservice
```

instead of using dynamic Pod IP addresses.

---

# Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-checkoutservice
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: checkoutservice
  app.kubernetes.io/name: opentelemetry-demo-checkoutservice
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

## Purpose

Labels provide metadata about the Service.

### Benefits

Labels help Kubernetes:

* Identify resources
* Group application components
* Support monitoring
* Simplify troubleshooting
* Enable filtering

---

# Service Type

```yaml
type: ClusterIP
```

## Purpose

Defines how the Service is exposed.

### What Is ClusterIP?

ClusterIP exposes the application only inside the Kubernetes cluster.

### Access Scope

```text
Inside Kubernetes Cluster  → Allowed
Outside Kubernetes Cluster → Not Allowed
```

### Why ClusterIP?

The Checkout Service is primarily consumed by internal microservices and the Frontend running inside the cluster.

---

# Port Configuration

```yaml
ports:
  - port: 8080
    name: tcp-service
    targetPort: 8080
```

This section controls how traffic reaches the application.

---

## Service Port

```yaml
port: 8080
```

### Purpose

Defines the port exposed by the Service.

Applications connect using:

```text
opentelemetry-demo-checkoutservice:8080
```

---

## Target Port

```yaml
targetPort: 8080
```

### Purpose

Defines the destination port inside the Pod.

### Traffic Flow

```text
Service Port 8080
        ↓
Target Port 8080
        ↓
Checkout Service Container
```

---

## Port Name

```yaml
name: tcp-service
```

### Purpose

Provides a readable identifier for the port.

### Benefits

* Easier debugging
* Easier monitoring
* Better service discovery

---

# Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-checkoutservice
```

## Purpose

Defines which Pods should receive traffic.

### How It Works

The Checkout Deployment creates Pods with:

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-checkoutservice
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
Maintains Desired Replicas
Restarts Failed Pods
Performs Rolling Updates
Supports Rollbacks
```

## Service Responsibilities

The Service:

```text
Provides Stable DNS Name
Provides Stable Network Endpoint
Routes Traffic To Pods
Load Balances Requests
Hides Pod IP Changes
```

---

# Why Service Is Needed

Without a Service:

```text
Frontend
    ↓
Pod IP (10.244.1.10)
```

Problem:

```text
Pod Restarts
      ↓
New Pod Created
      ↓
New IP Assigned
      ↓
Communication Breaks
```

---

# How Service Solves This Problem

With Service:

```text
Frontend
    ↓
opentelemetry-demo-checkoutservice
    ↓
Service Finds Current Pod
    ↓
Request Delivered
```

The DNS name remains constant even when Pods change.

---

# Checkout Request Flow

When a customer clicks **Place Order**:

```text
Frontend
    ↓
Checkout Service DNS
(opentelemetry-demo-checkoutservice)
    ↓
Kubernetes Service
    ↓
Checkout Service Pod
    ↓
Port 8080
    ↓
Checkout Process Started
```

---

# Internal Checkout Workflow

After receiving a request, the Checkout Service communicates with multiple backend services.

```text
Checkout Service
      ↓
Cart Service
      ↓
Product Catalog Service
      ↓
Currency Service
      ↓
Payment Service
      ↓
Shipping Service
      ↓
Kafka
      ↓
Email Service
      ↓
Order Completed
```

The Service acts as the entry point into this workflow.

---

# Load Balancing Scenario

Current Deployment:

```yaml
replicas: 1
```

Traffic Flow:

```text
Service
    ↓
Pod-1
```

If replicas increase:

```yaml
replicas: 3
```

Pods:

```text
Pod-1
Pod-2
Pod-3
```

Service automatically distributes traffic:

```text
Request 1 → Pod-1
Request 2 → Pod-2
Request 3 → Pod-3
Request 4 → Pod-1
```

No Service modification is required.

---

# Service Discovery

Kubernetes automatically creates DNS entries.

### Internal DNS Name

```text
opentelemetry-demo-checkoutservice
```

### Access Example

```text
http://opentelemetry-demo-checkoutservice:8080
```

Any Pod inside the cluster can use this address.

---

# Complete Runtime Flow

```text
kubectl apply deployment.yaml
            ↓
Checkout Service Pod Created
            ↓
Application Starts On Port 8080
            ↓

kubectl apply service.yaml
            ↓
Service Created
            ↓
Cluster IP Assigned
            ↓
Selector Finds Checkout Pods
            ↓
Stable DNS Name Created
            ↓
Frontend Sends Request
            ↓
Service Receives Request
            ↓
Traffic Routed To Checkout Pod
            ↓
Checkout Workflow Executed
            ↓
Response Returned
```

---

# Summary

This Service file:

* Creates a stable endpoint for Checkout Service
* Exposes the Checkout Service on port 8080
* Uses ClusterIP networking
* Makes the service available inside the Kubernetes cluster
* Automatically discovers Checkout Service Pods using labels
* Routes requests to running Pods
* Provides internal DNS-based service discovery
* Supports load balancing when multiple Pods exist
* Prevents failures caused by changing Pod IP addresses
* Acts as the entry point for the complete order placement workflow

The overall goal of this Service is to provide reliable and stable communication to the Checkout Service, ensuring that customer checkout requests can always reach the appropriate application Pods regardless of restarts, updates, or scaling operations.
