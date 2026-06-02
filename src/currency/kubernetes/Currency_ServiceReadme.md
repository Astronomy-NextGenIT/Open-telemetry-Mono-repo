# Currency Service Service Documentation

## Overview

This `service.yaml` file creates a Kubernetes **Service** for the Currency Service.

The Currency Service is responsible for currency conversion operations within the OpenTelemetry Demo application. Since Kubernetes Pods can be recreated and assigned new IP addresses, this Service provides a stable network endpoint that other services can use to communicate with the Currency Service.

The Service ensures reliable communication between microservices such as Checkout Service and Currency Service.

---

# High-Level Service Flow

```text
service.yaml
      ↓
Service Created
      ↓
Cluster IP Assigned
      ↓
Service Finds Currency Service Pods
      ↓
Traffic Routed To Pods
      ↓
Currency Service Accessible Inside Cluster
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

Kubernetes creates a stable networking endpoint for the Currency Service.

---

# Service Name

```yaml
metadata:
  name: opentelemetry-demo-currencyservice
```

## Purpose

Provides a stable DNS name for the Currency Service.

### Result

Other services can access Currency Service using:

```text
opentelemetry-demo-currencyservice
```

instead of using Pod IP addresses.

---

# Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-currencyservice
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: currencyservice
  app.kubernetes.io/name: opentelemetry-demo-currencyservice
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

## Purpose

Labels provide metadata about the Service.

### Benefits

Labels help Kubernetes:

* Identify resources
* Group components
* Support monitoring
* Enable filtering
* Simplify troubleshooting

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

The Currency Service is an internal backend service used by other microservices and does not need direct external access.

---

# Port Configuration

```yaml
ports:
  - port: 8080
    name: tcp-service
    targetPort: 8080
```

This section controls how requests reach the application.

---

## Service Port

```yaml
port: 8080
```

### Purpose

Defines the port exposed by the Service.

Applications communicate using:

```text
opentelemetry-demo-currencyservice:8080
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
Currency Service Container
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
  opentelemetry.io/name: opentelemetry-demo-currencyservice
```

## Purpose

Defines which Pods should receive traffic.

### How It Works

The Currency Service Deployment creates Pods with the following label:

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-currencyservice
```

The Service searches for Pods with the same label.

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

# Why Service Is Required

Without a Service:

```text
Checkout Service
       ↓
Pod IP (10.244.1.25)
```

Problem:

```text
Pod Restarts
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
Checkout Service
       ↓
opentelemetry-demo-currencyservice
       ↓
Service Finds Current Pod
       ↓
Request Delivered
```

The DNS name remains constant even when Pods are recreated.

---

# Currency Conversion Request Flow

Whenever a service needs currency conversion:

```text
Checkout Service
      ↓
Currency Service DNS
(opentelemetry-demo-currencyservice)
      ↓
Kubernetes Service
      ↓
Currency Service Pod
      ↓
Port 8080
      ↓
Currency Conversion
      ↓
Response Returned
```

---

# Example Conversion Flow

```text
Product Price = 100 USD
           ↓
Checkout Service
           ↓
Currency Service
           ↓
Convert USD To INR
           ↓
8300 INR
           ↓
Response Returned
```

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
opentelemetry-demo-currencyservice
```

### Access Example

```text
http://opentelemetry-demo-currencyservice:8080
```

Any Pod inside the cluster can use this address.

---

# Complete Runtime Flow

```text
kubectl apply deployment.yaml
            ↓
Currency Service Pod Created
            ↓
Application Starts On Port 8080
            ↓

kubectl apply service.yaml
            ↓
Service Created
            ↓
Cluster IP Assigned
            ↓
Selector Finds Currency Service Pod
            ↓
Stable DNS Name Created
            ↓
Checkout Service Sends Request
            ↓
Service Receives Request
            ↓
Traffic Routed To Currency Service Pod
            ↓
Currency Conversion Performed
            ↓
Response Returned
```

---

# Summary

This Service file:

* Creates a stable endpoint for Currency Service
* Exposes the Currency Service on port 8080
* Uses ClusterIP networking
* Makes the service available inside the Kubernetes cluster
* Automatically discovers Currency Service Pods using labels
* Routes requests to running Pods
* Provides internal DNS-based service discovery
* Supports load balancing when multiple Pods exist
* Prevents failures caused by changing Pod IP addresses
* Enables reliable currency conversion requests between microservices

The overall goal of this Service is to provide a stable and reliable communication layer for the Currency Service, ensuring that applications can perform currency conversions consistently regardless of Pod restarts, updates, or scaling operations.
