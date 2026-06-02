# Ad Service (Service Documentation)

## Overview

This `service.yaml` file creates a Kubernetes **Service** for the Ad Service.

The purpose of a Service is to provide a stable network endpoint for Pods. Since Pod IP addresses can change whenever Pods restart, 
Kubernetes Services provide a permanent way for other applications to communicate with the Ad Service.

Without a Service, other microservices would not know how to reliably connect to the Ad Service.

---

# High-Level Flow

```text
service.yaml
      ↓
Service Created
      ↓
Service Gets Cluster IP
      ↓
Service Finds Matching Pods
      ↓
Traffic Routed To Ad Service Pods
      ↓
Application Accessible Inside Cluster
```

---

# Resource Type

```yaml
apiVersion: v1
kind: Service
```

## Purpose

This tells Kubernetes:

* Use the Core API (`v1`)
* Create a Service resource

### What Happens

When executed:

```bash
kubectl apply -f service.yaml
```

Kubernetes creates a networking layer that routes requests to Ad Service Pods.

---

# Service Name

```yaml
metadata:
  name: opentelemetry-demo-adservice
```

## Purpose

Defines the Service name.

### Result

Other applications can communicate using:

```text
opentelemetry-demo-adservice
```

instead of using Pod IP addresses.

---

# Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-adservice
  app.kubernetes.io/component: adservice
  app.kubernetes.io/name: opentelemetry-demo-adservice
```

## Purpose

Labels help identify and organize Kubernetes resources.

### Benefits

* Easier monitoring
* Easier filtering
* Better resource management
* Consistent application grouping

---

# Service Type

```yaml
type: ClusterIP
```

## Purpose

Defines how the Service is exposed.

### What is ClusterIP?

`ClusterIP` exposes the application only inside the Kubernetes cluster.

### Access Scope

```text
Inside Kubernetes Cluster   → Allowed
Outside Kubernetes Cluster  → Not Allowed
```

### Usage

This is ideal for microservice-to-microservice communication.

---

# Port Configuration

```yaml
ports:
  - port: 8080
    name: tcp-service
    targetPort: 8080
```

This section defines how traffic is routed.

---

## Service Port

```yaml
port: 8080
```

### Purpose

Port exposed by the Service.

Other services call:

```text
opentelemetry-demo-adservice:8080
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
Ad Service Container
```

---

## Port Name

```yaml
name: tcp-service
```

### Purpose

Provides a human-readable name for the port.

### Benefits

* Easier debugging
* Easier service discovery
* Better observability integration

---

# Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-adservice
```

## Purpose

Tells the Service which Pods should receive traffic.

### How It Works

Kubernetes searches for Pods with:

```yaml
opentelemetry.io/name: opentelemetry-demo-adservice
```

The Deployment creates Pods with exactly the same label:

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-adservice
```

Because the labels match:

```text
Service
    ↓
Find Matching Pods
    ↓
Route Traffic
```

---

# Relationship Between Deployment and Service

## Deployment

Responsible for:

```text
Creating Pods
Managing Pods
Restarting Failed Pods
Scaling Pods
Rolling Updates
```

## Service

Responsible for:

```text
Finding Pods
Providing Stable Network Access
Load Balancing Traffic
```

---

# Complete Communication Flow

```text
Deployment
      ↓
Creates Ad Service Pod
      ↓
Pod Gets Dynamic IP
      ↓
Service Finds Pod Using Labels
      ↓
Service Creates Stable Endpoint
      ↓
Other Services Connect Using Service Name
      ↓
Traffic Routed To Pod
```

---

# Example Request Flow

Suppose the Frontend Service needs advertisements.

### Step 1

Frontend sends request:

```text
opentelemetry-demo-adservice:8080
```

### Step 2

Kubernetes Service receives request.

### Step 3

Service checks selector:

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-adservice
```

### Step 4

Matching Ad Service Pod is found.

### Step 5

Traffic forwarded:

```text
Frontend
      ↓
Ad Service Service
      ↓
Ad Service Pod
      ↓
Port 8080
      ↓
Application
```

---

# Why Service Is Important

Without Service:

```text
Frontend
      ↓
Pod IP (10.1.2.5)
```

Problem:

```text
Pod Restart
      ↓
New IP Assigned
      ↓
Frontend Connection Breaks
```

With Service:

```text
Frontend
      ↓
opentelemetry-demo-adservice
      ↓
Service Finds New Pod Automatically
```

No application changes are required.

---

# Complete Runtime Flow

```text
kubectl apply deployment.yaml
            ↓
Deployment Created
            ↓
Pod Created
            ↓
Ad Service Starts On Port 8080
            ↓

kubectl apply service.yaml
            ↓
Service Created
            ↓
Selector Finds Ad Service Pod
            ↓
Cluster IP Assigned
            ↓
Stable Service Endpoint Created
            ↓
Other Microservices Access:
opentelemetry-demo-adservice:8080
            ↓
Traffic Routed To Pod
            ↓
Response Returned
```

---

# Summary

This Service file:

* Creates a stable network endpoint for Ad Service
* Exposes port 8080 inside the Kubernetes cluster
* Uses ClusterIP networking
* Automatically discovers Pods using labels
* Routes traffic to Ad Service containers
* Provides service discovery using DNS
* Supports load balancing when multiple Pods exist
* Prevents communication failures caused by changing Pod IPs

The overall goal of this Service is to make the Ad Service reliably accessible to other microservices running inside the Kubernetes cluster.
