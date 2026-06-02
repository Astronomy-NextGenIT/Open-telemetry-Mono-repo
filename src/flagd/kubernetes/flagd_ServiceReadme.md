# Flagd Service Documentation

## Overview

This `service.yaml` file creates a Kubernetes **Service** for the Flagd Deployment.

Unlike most services in the OpenTelemetry Demo, this Service exposes **two different ports** because the Flagd Pod contains **two containers**:

1. **Flagd Service** (Feature Flag Engine) – Port 8013
2. **Flagd UI** (Web Management Interface) – Port 4000

This Service provides stable network access to both containers and enables other microservices to communicate with Flagd for feature flag evaluation.

---

# High-Level Service Flow

```text
service.yaml
      ↓
Service Created
      ↓
Cluster IP Assigned
      ↓
Service Finds Flagd Pod
      ↓
Routes Traffic To Containers
      ↓
Flagd Accessible On Port 8013
      ↓
Flagd UI Accessible On Port 4000
```

---

# Service Resource

```yaml
apiVersion: v1
kind: Service
```

## Purpose

Creates a Kubernetes Service resource.

### What Happens

```text
kubectl apply service.yaml
          ↓
Service Created
          ↓
Stable DNS Name Assigned
          ↓
Traffic Routing Enabled
```

---

# Service Name

```yaml
metadata:
  name: opentelemetry-demo-flagd
```

## Purpose

Creates a stable DNS name for Flagd.

### DNS Name

```text
opentelemetry-demo-flagd
```

Applications use this name instead of Pod IP addresses.

---

# Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-flagd
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: flagd
  app.kubernetes.io/name: opentelemetry-demo-flagd
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

## Purpose

Labels provide metadata for:

* Resource identification
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

Exposes Flagd only inside the Kubernetes cluster.

### Access Scope

```text
Inside Cluster  → Allowed
Outside Cluster → Not Allowed
```

### Why ClusterIP?

Flagd is an internal service used by microservices and developers inside the cluster.

---

# Port Configuration

This Service exposes **two ports**.

---

# Port 1 – Flagd Engine

```yaml
ports:
  - port: 8013
    name: tcp-service
    targetPort: 8013
```

## Purpose

Provides access to the Feature Flag Engine.

### Traffic Flow

```text
Application
      ↓
Service Port 8013
      ↓
Target Port 8013
      ↓
Flagd Container
```

### Used By

Services such as:

* Cart Service
* Checkout Service
* Product Catalog Service
* Ad Service

These services query Flagd to determine whether a feature is enabled or disabled.

---

# Example Feature Flag Request

```text
Cart Service
      ↓
opentelemetry-demo-flagd:8013
      ↓
Feature Flag Check
      ↓
Enabled / Disabled Response
```

---

# Port 2 – Flagd UI

```yaml
- port: 4000
  name: tcp-service-0
  targetPort: 4000
```

## Purpose

Provides access to the Flagd Web UI.

### Traffic Flow

```text
Browser
    ↓
Service Port 4000
    ↓
Target Port 4000
    ↓
Flagd UI Container
```

---

# Why Two Ports?

The Deployment contains two containers:

```text
Flagd Pod
│
├── Flagd Container
│      Port 8013
│
└── Flagd UI Container
       Port 4000
```

The Service exposes both containers through a single Service resource.

---

# Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-flagd
```

## Purpose

Identifies which Pods should receive traffic.

### How It Works

The Deployment creates Pods with:

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-flagd
```

The Service searches for Pods containing the same label.

### Matching Process

```text
Service Selector
        ↓
Find Matching Pod
        ↓
Attach Pod To Service
        ↓
Route Traffic
```

---

# Relationship Between Deployment and Service

## Deployment Responsibilities

The Deployment:

```text
Creates Pod
Maintains Replica Count
Restarts Failed Pod
Performs Updates
Handles Rollbacks
```

## Service Responsibilities

The Service:

```text
Provides Stable DNS
Provides Stable Endpoint
Routes Requests
Load Balances Traffic
Hides Pod IP Changes
```

---

# Why Service Is Needed

Without a Service:

```text
Application
      ↓
Pod IP (10.244.1.35)
```

Problem:

```text
Pod Restart
     ↓
New Pod Created
     ↓
New IP Assigned
     ↓
Applications Lose Connection
```

---

# How Service Solves This Problem

With Service:

```text
Application
      ↓
opentelemetry-demo-flagd
      ↓
Service Finds Running Pod
      ↓
Request Delivered
```

The DNS name remains constant.

---

# Feature Flag Runtime Flow

```text
Cart Service
      ↓
Feature Check
      ↓
opentelemetry-demo-flagd:8013
      ↓
Flagd Reads Configuration
      ↓
Feature Status Returned
      ↓
Application Adjusts Behavior
```

---

# Flagd UI Runtime Flow

```text
Administrator
      ↓
Flagd UI
      ↓
opentelemetry-demo-flagd:4000
      ↓
View Feature Flags
      ↓
Modify Configuration
      ↓
Feature Updates Applied
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

Traffic Distribution:

```text
Request 1 → Pod-1
Request 2 → Pod-2
Request 3 → Pod-3
Request 4 → Pod-1
```

Kubernetes automatically balances traffic.

---

# Service Discovery

Kubernetes automatically creates DNS records.

### Internal DNS Name

```text
opentelemetry-demo-flagd
```

### Access Examples

Flagd Engine:

```text
opentelemetry-demo-flagd:8013
```

Flagd UI:

```text
opentelemetry-demo-flagd:4000
```

Any Pod inside the cluster can use these addresses.

---

# Complete Runtime Flow

```text
kubectl apply deployment.yaml
            ↓
Flagd Pod Created
            ↓
Flagd Container Starts
            ↓
Flagd UI Container Starts
            ↓

kubectl apply service.yaml
            ↓
Service Created
            ↓
Cluster IP Assigned
            ↓
Selector Finds Flagd Pod
            ↓
DNS Name Created
            ↓
Port 8013 Exposed
            ↓
Port 4000 Exposed
            ↓
Feature Flag Service Ready
```

---

# Summary

This Service file:

* Creates a stable endpoint for Flagd
* Uses ClusterIP networking
* Exposes Flagd Engine on port 8013
* Exposes Flagd UI on port 4000
* Routes traffic to the correct container
* Provides internal DNS-based service discovery
* Automatically discovers Flagd Pods using labels
* Supports load balancing when multiple replicas exist
* Prevents communication failures caused by changing Pod IPs
* Enables centralized feature flag management for all microservices

The overall goal of this Service is to provide reliable access to both the Feature Flag Engine and the Flag Management UI, allowing applications to dynamically enable or disable features without requiring code changes or redeployments.
