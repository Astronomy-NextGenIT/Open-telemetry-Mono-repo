# Currency Service Deployment Documentation

## Overview

This `deploy.yaml` file is responsible for deploying the **Currency Service** in the Kubernetes cluster.

The Currency Service handles currency conversion operations within the OpenTelemetry Demo application. Whenever a service needs to convert product prices from one currency 
to another (for example, USD to INR or EUR to USD), it communicates with the Currency Service.

This Deployment ensures the Currency Service is always running, monitored, restarted automatically if it fails, and integrated with OpenTelemetry for observability.

---

# High-Level Architecture

```text
Frontend
    ↓
Checkout Service
    ↓
Currency Service
    ↓
Currency Conversion
    ↓
Converted Amount Returned
```

---

# Deployment Resource

```yaml
apiVersion: apps/v1
kind: Deployment
```

## Purpose

Defines a Kubernetes Deployment resource.

### What Happens

```text
kubectl apply deploy.yaml
            ↓
Deployment Created
            ↓
ReplicaSet Created
            ↓
Pod Created
            ↓
Currency Service Started
```

The Deployment continuously ensures the application remains available.

---

# Deployment Name

```yaml
metadata:
  name: opentelemetry-demo-currencyservice
```

## Purpose

Provides a unique identifier for the Currency Service deployment.

### Usage

Used by Kubernetes for:

* Resource management
* Monitoring
* Updates
* Rollbacks

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

Labels provide metadata for resource identification.

### Benefits

* Resource grouping
* Pod selection
* Service discovery
* Monitoring
* Troubleshooting

---

# Replica Configuration

```yaml
replicas: 1
```

## Purpose

Ensures one Currency Service Pod is always running.

### Failure Recovery

```text
Pod Fails
    ↓
Deployment Detects Failure
    ↓
New Pod Created
```

---

# Revision History

```yaml
revisionHistoryLimit: 10
```

## Purpose

Stores up to 10 previous Deployment versions.

### Benefit

Allows rollback to a previous working version.

---

# Selector

```yaml
selector:
  matchLabels:
    opentelemetry.io/name: opentelemetry-demo-currencyservice
```

## Purpose

Defines which Pods belong to this Deployment.

### Flow

```text
Deployment
      ↓
Find Matching Pods
      ↓
Manage Pods
```

---

# Pod Template

```yaml
template:
```

## Purpose

Acts as a blueprint for creating Currency Service Pods.

Whenever Kubernetes creates a new Pod, it uses this template.

---

# Service Account

```yaml
serviceAccountName: opentelemetry-demo
```

## Purpose

Associates the Pod with a Kubernetes Service Account.

### Benefit

Provides controlled Kubernetes permissions.

---

# Main Application Container

```yaml
containers:
  - name: currencyservice
```

## Purpose

Runs the Currency Service application.

---

# Container Image

```yaml
image: ghcr.io/open-telemetry/demo:1.12.0-currencyservice
```

## Purpose

Specifies the Docker image used to run the service.

### Startup Flow

```text
Node
 ↓
Pull Image
 ↓
Create Container
 ↓
Start Currency Service
```

---

# Image Pull Policy

```yaml
imagePullPolicy: IfNotPresent
```

## Purpose

Controls image download behavior.

### Behavior

```text
Image Exists
      ↓
Use Local Copy

Image Missing
      ↓
Download From Registry
```

---

# Container Port

```yaml
ports:
  - containerPort: 8080
```

## Purpose

Exposes port 8080 inside the container.

### What Happens

The Currency Service listens on:

```text
Port 8080
```

for incoming requests.

---

# Environment Variables

Environment variables provide runtime configuration.

---

## OTEL_SERVICE_NAME

```yaml
OTEL_SERVICE_NAME
```

### Purpose

Automatically retrieves the service name from Pod labels.

### Result

```text
currencyservice
```

This value appears in traces, metrics, and logs.

---

## OTEL_COLLECTOR_NAME

```yaml
OTEL_COLLECTOR_NAME=opentelemetry-demo-otelcol
```

### Purpose

Defines the OpenTelemetry Collector service responsible for receiving telemetry data.

---

## Metrics Temporality

```yaml
OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE=cumulative
```

### Purpose

Controls how metrics are reported.

### Meaning

```text
Metrics accumulate over time.
```

This provides long-term monitoring data.

---

## Currency Service Port

```yaml
CURRENCY_SERVICE_PORT=8080
```

### Purpose

Defines the port on which the application listens.

### Result

```text
Currency Service
      ↓
Port 8080
```

---

## Version Information

```yaml
VERSION=1.12.0
```

### Purpose

Defines the application version.

### Benefits

* Easier troubleshooting
* Version tracking
* Observability correlation

### Example

```text
Currency Service Version
          ↓
1.12.0
```

---

## OpenTelemetry Endpoint

```yaml
OTEL_EXPORTER_OTLP_ENDPOINT=http://opentelemetry-demo-otelcol:4317
```

## Purpose

Defines where telemetry data should be exported.

### Data Sent

```text
Traces
Metrics
Logs
```

### Flow

```text
Currency Service
        ↓
OpenTelemetry Collector
        ↓
Observability Platform
```

---

## Resource Attributes

```yaml
OTEL_RESOURCE_ATTRIBUTES
```

## Purpose

Adds metadata to telemetry records.

### Example

```text
service.name=currencyservice
service.namespace=opentelemetry-demo
service.version=1.12.0
```

This metadata helps identify the service inside monitoring tools.

---

# Currency Conversion Workflow

When another service requests currency conversion:

```text
Checkout Service
      ↓
Currency Service
      ↓
Convert Amount
      ↓
Return Converted Value
```

Example:

```text
100 USD
   ↓
Currency Service
   ↓
8300 INR
```

---

# Resource Limits

```yaml
resources:
  limits:
    memory: 20Mi
```

## Purpose

Restricts memory consumption.

### Maximum Allowed

```text
20 MiB
```

### Benefit

Prevents excessive resource usage.

---

# Volume Mounts

```yaml
volumeMounts:
```

## Purpose

Used to mount storage into containers.

### Current State

No volume mounts are configured.

---

# Volumes

```yaml
volumes:
```

## Purpose

Defines storage resources available to the Pod.

### Current State

No storage volumes are configured.

These sections are reserved for future requirements such as:

* Persistent Volumes
* Config Files
* Secrets
* Shared Storage

---

# Complete Deployment Flow

```text
kubectl apply deploy.yaml
            ↓
Deployment Created
            ↓
ReplicaSet Created
            ↓
Pod Created
            ↓
Container Image Pulled
            ↓
Currency Service Started
            ↓
Port 8080 Opened
            ↓
Telemetry Configured
            ↓
Connected To OpenTelemetry Collector
            ↓
Currency Service Ready
```

---

# Runtime Request Flow

```text
Checkout Service
      ↓
Currency Service
      ↓
Currency Conversion
      ↓
Converted Value Generated
      ↓
Response Returned
```

---

# Summary

This Deployment file:

* Deploys the Currency Service application
* Maintains one running Pod
* Exposes application port 8080
* Supports currency conversion requests
* Integrates with OpenTelemetry
* Sends traces, metrics, and logs to the Collector
* Automatically recreates failed Pods
* Supports rolling updates and rollbacks
* Tracks application version
* Restricts memory usage to 20Mi
* Uses Kubernetes labels for resource identification

The overall goal of this Deployment is to provide a reliable currency conversion service for the OpenTelemetry Demo application while ensuring observability, scalability, and operational stability.
