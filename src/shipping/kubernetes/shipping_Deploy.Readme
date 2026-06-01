# Shipping Service Deployment (`deploy.yaml`) Documentation

## Overview

This Kubernetes Deployment file deploys the **Shipping Service** in the OpenTelemetry Demo application.

The Shipping Service is responsible for calculating shipping costs and shipping options during the checkout process. When a customer places an order, this service communicates with the Quote Service to obtain shipping quotes and returns shipping information to the Checkout Service.

For example:

```text
Customer Places Order
         |
         v
Shipping Service
         |
         v
Quote Service
         |
         v
Shipping Cost Calculated
```

This service plays an important role in completing customer orders by providing delivery-related information.

---

# Shipping Service Architecture

```text
Customer
   |
   v
Checkout Service
   |
   v
Shipping Service
   |
   v
Quote Service
   |
   v
Shipping Cost
```

The Shipping Service depends on the Quote Service to generate shipping quotes.

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

* Automatic Pod creation
* Self-healing
* Rolling updates
* Rollback capability

---

# 2. Metadata

```yaml
metadata:
  name: opentelemetry-demo-shippingservice
```

## Purpose

Defines the Deployment name:

```text
opentelemetry-demo-shippingservice
```

Kubernetes uses this name to manage Shipping Service Pods.

---

# 3. Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-shippingservice
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: shippingservice
  app.kubernetes.io/name: opentelemetry-demo-shippingservice
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

## Purpose

Labels help Kubernetes:

* Identify resources
* Group related components
* Enable monitoring
* Support service discovery
* Select Pods

### Example

```yaml
app.kubernetes.io/component: shippingservice
```

Indicates this component belongs to the Shipping Service.

---

# 4. Deployment Specification

```yaml
spec:
```

Defines how the Shipping Service should run.

---

## 4.1 Replicas

```yaml
replicas: 1
```

### Purpose

Runs one Shipping Service Pod.

```text
Shipping Deployment
         |
         v
Shipping Pod
```

If the Pod fails, Kubernetes automatically recreates it.

---

## 4.2 Revision History

```yaml
revisionHistoryLimit: 10
```

### Purpose

Stores the last 10 deployment versions.

Allows rollback to a previous version if needed.

---

# 5. Selector

```yaml
selector:
  matchLabels:
    opentelemetry.io/name: opentelemetry-demo-shippingservice
```

## Purpose

The Deployment manages Pods that contain this label.

---

# 6. Pod Template

```yaml
template:
```

Defines how Pods are created.

---

## Pod Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-shippingservice
```

Applied automatically to all Shipping Service Pods.

---

# 7. Service Account

```yaml
serviceAccountName: opentelemetry-demo
```

## Purpose

The Pod uses the Service Account:

```text
opentelemetry-demo
```

This provides controlled access to Kubernetes resources.

---

# 8. Shipping Service Container

```yaml
containers:
  - name: shippingservice
```

Main application container.

---

## 8.1 Container Image

```yaml
image: ghcr.io/open-telemetry/demo:1.12.0-shippingservice
```

### Breakdown

```text
Registry : ghcr.io
Project  : open-telemetry/demo
Version  : 1.12.0
Service  : shippingservice
```

### Purpose

Contains the Shipping Service application code.

---

## 8.2 Image Pull Policy

```yaml
imagePullPolicy: IfNotPresent
```

### Purpose

* Uses local image if available
* Downloads image only when necessary

### Benefit

Reduces startup time and network usage.

---

# 9. Container Port

```yaml
ports:
  - containerPort: 8080
```

## Purpose

The Shipping Service listens on:

```text
8080
```

Other services communicate with it through this port.

---

# 10. OpenTelemetry Configuration

## Service Name

```yaml
OTEL_SERVICE_NAME
```

Obtained dynamically from Pod labels.

Resolved value:

```text
shippingservice
```

### Purpose

Used to identify the service in:

* Traces
* Metrics
* Logs

---

## Collector Name

```yaml
OTEL_COLLECTOR_NAME=opentelemetry-demo-otelcol
```

### Purpose

Defines the OpenTelemetry Collector service.

---

## Metrics Temporality

```yaml
OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE=cumulative
```

### Purpose

Exports cumulative metrics.

---

# 11. Shipping Service Port

```yaml
SHIPPING_SERVICE_PORT=8080
```

## Purpose

Configures the application to listen on port 8080.

```text
Shipping Service
        |
        v
      8080
```

---

# 12. Quote Service Connection

```yaml
QUOTE_SERVICE_ADDR=http://opentelemetry-demo-quoteservice:8080
```

## Purpose

Allows the Shipping Service to communicate with the Quote Service.

### Workflow

```text
Shipping Service
        |
        v
Quote Service
        |
        v
Shipping Cost Calculation
```

The Quote Service provides shipping quotes that are returned to customers during checkout.

---

## Example Flow

```text
Customer Places Order
         |
         v
Checkout Service
         |
         v
Shipping Service
         |
         v
Quote Service
         |
         v
Shipping Cost
         |
         v
Customer Sees Delivery Charges
```

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

Exports:

* Traces
* Metrics
* Logs

to the OpenTelemetry Collector.

---

# 14. Resource Attributes

```yaml
OTEL_RESOURCE_ATTRIBUTES
```

Resolved value:

```text
service.name=shippingservice,
service.namespace=opentelemetry-demo,
service.version=1.12.0
```

### Purpose

Adds metadata to telemetry data.

Used by:

* Jaeger
* Prometheus
* Grafana
* OpenTelemetry Collector

---

# 15. Resource Limits

```yaml
resources:
  limits:
    memory: 20Mi
```

## Purpose

Limits memory consumption to:

```text
20 MiB
```

### Why So Small?

The Shipping Service performs lightweight operations:

* Receives requests
* Calls Quote Service
* Returns shipping costs

It does not perform heavy computations or maintain large datasets.

---

# 16. Volume Mounts

```yaml
volumeMounts:
```

No volume mounts are configured.

### Meaning

The application runs entirely from the container image.

---

# 17. Volumes

```yaml
volumes:
```

No volumes are defined.

### Meaning

No persistent storage is required.

---

# Shipping Workflow

```text
Customer Checkout
        |
        v
Checkout Service
        |
        v
Shipping Service
        |
        v
Quote Service
        |
        v
Calculate Shipping Cost
        |
        v
Return Shipping Information
```

---

# Observability Flow

```text
Shipping Service
        |
        v
OpenTelemetry Collector
        |
        +--> Jaeger
        +--> Prometheus
        +--> Grafana
```

All shipping-related requests are monitored through OpenTelemetry.

---

# Deployment Startup Flow

```text
Deployment Created
        |
        v
Pod Created
        |
        v
Container Started
        |
        v
Listening on Port 8080
        |
        v
Connected to Quote Service
        |
        v
Connected to OTEL Collector
        |
        v
Ready to Process Shipping Requests
```

---

# Architecture Diagram

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
Shipping Service
   |
   v
Quote Service
```

---

# Summary

This Deployment:

* Deploys the Shipping Service.
* Uses image `ghcr.io/open-telemetry/demo:1.12.0-shippingservice`.
* Runs one Pod.
* Listens on port `8080`.
* Connects to the Quote Service.
* Sends telemetry to the OpenTelemetry Collector.
* Uses only `20Mi` of memory.
* Supports self-healing and rolling updates.
* Participates in the checkout and shipping workflow.

## In Simple Terms

The Shipping Service is the **delivery cost calculator** of the OpenTelemetry Demo application. During checkout, it communicates with the Quote Service to determine shipping costs and delivery information. It then returns this information to the checkout process so customers can see shipping charges before placing an order. All operations are monitored using OpenTelemetry, making troubleshooting and performance analysis easy.
