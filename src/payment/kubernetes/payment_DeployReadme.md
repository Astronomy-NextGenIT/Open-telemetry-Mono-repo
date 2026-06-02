# Payment Service Deployment (`deploy.yaml`) Documentation

## Overview

This Kubernetes Deployment file deploys the **Payment Service** in the OpenTelemetry Demo application.

The Payment Service is responsible for **processing customer payments** during the checkout process. When a customer places an order, the Checkout Service sends payment 
details to the Payment Service, which simulates payment authorization and returns the result.

In the OpenTelemetry Demo, the Payment Service does not connect to a real bank or payment gateway. Instead, it simulates payment processing so developers can observe traces
, metrics, and logs across microservices.

---

# Payment Service Architecture

```text
Customer
    |
    v
Frontend
    |
    v
Checkout Service
    |
    v
Payment Service
    |
    v
Payment Approved / Rejected
    |
    v
Checkout Continues
```

The Payment Service is a critical part of the order processing workflow.

---

# 1. API Version and Resource Type

```yaml
apiVersion: apps/v1
kind: Deployment
```

## Purpose

### apiVersion

Uses the Kubernetes Deployment API.

### kind

Creates a Deployment resource.

### Benefits

* Self-healing
* Rolling updates
* Rollbacks
* Automatic Pod management

---

# 2. Metadata

```yaml
metadata:
  name: opentelemetry-demo-paymentservice
```

## Purpose

Deployment name:

```text
opentelemetry-demo-paymentservice
```

Kubernetes uses this name to manage the Payment Service Deployment.

---

# 3. Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-paymentservice
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: paymentservice
  app.kubernetes.io/name: opentelemetry-demo-paymentservice
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

## Purpose

Labels help Kubernetes:

* Identify resources
* Select Pods
* Organize components
* Enable monitoring

### Example

```yaml
app.kubernetes.io/component: paymentservice
```

Indicates that this resource belongs to the Payment Service.

---

# 4. Deployment Specification

```yaml
spec:
```

Defines how the Payment Service should run.

---

## 4.1 Replicas

```yaml
replicas: 1
```

### Purpose

Runs one Payment Service Pod.

```text
Payment Service Deployment
            |
            v
     Payment Service Pod
```

If the Pod crashes, Kubernetes automatically creates a replacement.

---

## 4.2 Revision History

```yaml
revisionHistoryLimit: 10
```

### Purpose

Stores the last 10 deployment versions.

This allows rollback if a deployment update causes issues.

---

# 5. Selector

```yaml
selector:
  matchLabels:
    opentelemetry.io/name: opentelemetry-demo-paymentservice
```

## Purpose

The Deployment manages Pods that have this label.

---

# 6. Pod Template

```yaml
template:
```

Defines how Payment Service Pods should be created.

---

## Pod Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-paymentservice
```

These labels are automatically applied to every Payment Service Pod.

---

# 7. Service Account

```yaml
serviceAccountName: opentelemetry-demo
```

## Purpose

The Pod runs using the Service Account:

```text
opentelemetry-demo
```

This provides secure access to Kubernetes resources if required.

---

# 8. Payment Service Container

```yaml
containers:
  - name: paymentservice
```

This is the main container running the Payment Service application.

---

## 8.1 Container Image

```yaml
image: ghcr.io/open-telemetry/demo:1.12.0-paymentservice
```

### Breakdown

```text
Registry : ghcr.io
Project  : open-telemetry/demo
Version  : 1.12.0
Service  : paymentservice
```

### Purpose

Contains the Payment Service application code.

---

## 8.2 Image Pull Policy

```yaml
imagePullPolicy: IfNotPresent
```

### Purpose

* Uses local image if available
* Downloads image only when necessary

### Benefit

Faster startup and reduced network traffic.

---

# 9. Container Port

```yaml
ports:
  - containerPort: 8080
```

## Purpose

The Payment Service listens on:

```text
Port 8080
```

This port is used by the Checkout Service to communicate with the Payment Service.

---

# 10. OpenTelemetry Configuration

## Service Name

```yaml
OTEL_SERVICE_NAME
```

Resolved value:

```text
paymentservice
```

### Purpose

Identifies this service in traces, logs, and metrics.

---

## Collector Name

```yaml
OTEL_COLLECTOR_NAME=opentelemetry-demo-otelcol
```

### Purpose

Specifies the OpenTelemetry Collector.

---

## Metrics Temporality

```yaml
OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE=cumulative
```

### Purpose

Exports metrics as cumulative values.

---

# 11. Payment Service Port Configuration

```yaml
PAYMENT_SERVICE_PORT=8080
```

## Purpose

Defines the port on which the Payment Service listens.

```text
Payment Service
       |
       v
Port 8080
```

This value must match the container port.

---

# 12. Feature Flag Configuration

## Flagd Host

```yaml
FLAGD_HOST=opentelemetry-demo-flagd
```

## Flagd Port

```yaml
FLAGD_PORT=8013
```

### Purpose

Connects the Payment Service to the Flagd feature flag system.

Feature flags allow functionality to be enabled or disabled without redeploying the application.

Examples:

* Simulate payment failures
* Enable experimental payment behavior
* Test observability scenarios

---

# 13. Telemetry Export Endpoint

```yaml
OTEL_EXPORTER_OTLP_ENDPOINT=http://$(OTEL_COLLECTOR_NAME):4317
```

Resolved value:

```text
http://opentelemetry-demo-otelcol:4317
```

### Purpose

Exports telemetry data to the OpenTelemetry Collector.

Telemetry includes:

* Traces
* Metrics
* Logs

---

# 14. Resource Attributes

```yaml
OTEL_RESOURCE_ATTRIBUTES
```

Resolved value:

```text
service.name=paymentservice,
service.namespace=opentelemetry-demo,
service.version=1.12.0
```

### Purpose

Adds metadata to telemetry.

This information appears in:

* Grafana
* Jaeger
* OpenTelemetry Collector

---

# 15. Resource Limits

```yaml
resources:
  limits:
    memory: 120Mi
```

## Purpose

Limits maximum memory usage.

```text
120 MiB
```

### Benefit

Prevents the Payment Service from consuming excessive cluster memory.

---

# 16. Security Context

```yaml
securityContext:
  runAsGroup: 1000
  runAsNonRoot: true
  runAsUser: 1000
```

## Purpose

Runs the container securely.

### runAsNonRoot

```yaml
runAsNonRoot: true
```

Prevents execution as the root user.

---

### runAsUser

```yaml
runAsUser: 1000
```

Runs the application as User ID 1000.

---

### runAsGroup

```yaml
runAsGroup: 1000
```

Runs the application as Group ID 1000.

---

### Benefit

Reduces security risks if the container is compromised.

---

# 17. Volume Mounts

```yaml
volumeMounts:
```

No volume mounts are configured.

### Meaning

The Payment Service does not require persistent storage.

---

# 18. Volumes

```yaml
volumes:
```

No volumes are defined.

### Meaning

The application runs entirely from the container image.

---

# Payment Processing Flow

```text
Customer Places Order
          |
          v
Checkout Service
          |
          v
Payment Service
          |
          +--> Validate Payment
          |
          +--> Simulate Authorization
          |
          v
Payment Result
          |
          v
Checkout Service
```

---

# Observability Flow

```text
Payment Service
        |
        v
OpenTelemetry Collector
        |
        +--> Jaeger (Traces)
        +--> Prometheus (Metrics)
        +--> Grafana (Dashboards)
```

Every payment request generates telemetry data.

---

# Complete Application Flow

```text
User
 |
 v
Frontend
 |
 v
Checkout Service
 |
 v
Payment Service
 |
 v
Payment Approved
 |
 v
Shipping Service
 |
 v
Order Completed
```

---

# Deployment Startup Flow

```text
Deployment Created
        |
        v
Payment Service Pod Created
        |
        v
Container Started
        |
        v
Listening on Port 8080
        |
        v
Connected to Flagd
        |
        v
Connected to OTEL Collector
        |
        v
Ready for Payment Requests
```

---

# Summary

This Deployment:

* Deploys the Payment Service.
* Uses image `ghcr.io/open-telemetry/demo:1.12.0-paymentservice`.
* Listens on port `8080`.
* Simulates payment processing.
* Integrates with Flagd for feature flags.
* Sends telemetry to the OpenTelemetry Collector.
* Limits memory usage to `120Mi`.
* Runs as a non-root user (`UID 1000`).
* Supports self-healing and rolling updates.

## In Simple Terms

The Payment Service is responsible for handling payment operations during checkout. When a customer places an order, the Checkout Service calls the Payment Service to process the payment. The service simulates payment approval or failure, generates telemetry data for observability, and helps demonstrate distributed tracing and monitoring across the OpenTelemetry Demo microservices architecture.
