# Email Service (Service Documentation)

## Overview

This `service.yaml` file creates a Kubernetes **Service** for the Email Service.

The Email Service is responsible for generating and sending email notifications within the OpenTelemetry Demo application. When a customer completes an order, 
the Checkout Service communicates with the Email Service through this Service.

Since Kubernetes Pods can be recreated at any time and receive new IP addresses, the Service provides a stable network endpoint that other applications can always use.

---

# High-Level Service Flow

```text
service.yaml
      ↓
Service Created
      ↓
Cluster IP Assigned
      ↓
Service Finds Email Service Pods
      ↓
Traffic Routed To Pods
      ↓
Email Service Accessible Inside Cluster
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

Kubernetes creates a stable networking endpoint for the Email Service.

---

# Service Name

```yaml
metadata:
  name: opentelemetry-demo-emailservice
```

## Purpose

Provides a stable DNS name for the Email Service.

### Result

Other services can communicate using:

```text
opentelemetry-demo-emailservice
```

instead of using Pod IP addresses.

---

# Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-emailservice
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: emailservice
  app.kubernetes.io/name: opentelemetry-demo-emailservice
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

ClusterIP exposes the application only within the Kubernetes cluster.

### Access Scope

```text
Inside Kubernetes Cluster  → Allowed
Outside Kubernetes Cluster → Not Allowed
```

### Why ClusterIP?

The Email Service is an internal microservice used by other backend services and does not require direct external access.

---

# Port Configuration

```yaml
ports:
  - port: 8080
    name: tcp-service
    targetPort: 8080
```

This section controls how traffic reaches the Email Service.

---

## Service Port

```yaml
port: 8080
```

### Purpose

Defines the port exposed by the Service.

Applications connect using:

```text
opentelemetry-demo-emailservice:8080
```

---

## Target Port

```yaml
targetPort: 8080
```

### Purpose

Defines the port inside the Email Service Pod that receives traffic.

### Traffic Flow

```text
Service Port 8080
        ↓
Target Port 8080
        ↓
Email Service Container
```

---

## Port Name

```yaml
name: tcp-service
```

### Purpose

Provides a readable identifier for the port.

### Benefits

* Easier monitoring
* Easier troubleshooting
* Better service discovery

---

# Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-emailservice
```

## Purpose

Defines which Pods should receive traffic.

### How It Works

The Email Service Deployment creates Pods with the label:

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-emailservice
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
Pod IP (10.244.1.50)
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
Checkout Service
       ↓
opentelemetry-demo-emailservice
       ↓
Service Finds Current Pod
       ↓
Request Delivered
```

The DNS name remains unchanged even when Pods are recreated.

---

# Email Notification Workflow

When a customer places an order:

```text
Customer Places Order
          ↓
Checkout Service
          ↓
Email Service DNS
(opentelemetry-demo-emailservice)
          ↓
Kubernetes Service
          ↓
Email Service Pod
          ↓
Generate Email
          ↓
Send Confirmation Notification
          ↓
Success Response Returned
```

---

# Example Communication Flow

```text
Order Completed
      ↓
Checkout Service
      ↓
opentelemetry-demo-emailservice:8080
      ↓
Email Service
      ↓
Create Order Confirmation
      ↓
Send Email
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

No Service changes are required.

---

# Service Discovery

Kubernetes automatically creates DNS entries.

### Internal DNS Name

```text
opentelemetry-demo-emailservice
```

### Access Example

```text
http://opentelemetry-demo-emailservice:8080
```

Any Pod inside the cluster can use this address.

---

# Complete Runtime Flow

```text
kubectl apply deployment.yaml
            ↓
Email Service Pod Created
            ↓
Application Starts On Port 8080
            ↓

kubectl apply service.yaml
            ↓
Service Created
            ↓
Cluster IP Assigned
            ↓
Selector Finds Email Service Pod
            ↓
Stable DNS Name Created
            ↓
Checkout Service Sends Request
            ↓
Service Receives Request
            ↓
Traffic Routed To Email Service Pod
            ↓
Email Generated
            ↓
Notification Sent
            ↓
Response Returned
```

---

# Summary

This Service file:

* Creates a stable endpoint for Email Service
* Exposes the Email Service on port 8080
* Uses ClusterIP networking
* Makes the service available inside the Kubernetes cluster
* Automatically discovers Email Service Pods using labels
* Routes requests to running Pods
* Provides internal DNS-based service discovery
* Supports load balancing when multiple Pods exist
* Prevents failures caused by changing Pod IP addresses
* Enables reliable communication between Checkout Service and Email Service

The overall goal of this Service is to provide a stable and reliable communication layer for the Email Service, ensuring that order confirmation and notification requests can always reach the correct Email Service Pod regardless of restarts, updates, or scaling operations.
