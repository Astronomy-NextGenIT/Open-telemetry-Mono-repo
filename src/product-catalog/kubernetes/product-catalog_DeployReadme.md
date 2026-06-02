# Product Catalog Service Deployment (`deploy.yaml`) Documentation

## Overview

This Kubernetes Deployment file deploys the **Product Catalog Service** in the OpenTelemetry Demo application.

The Product Catalog Service is responsible for storing and serving product information to the application. Whenever a user visits the online store, searches products, or views product details, the Frontend communicates with this service to retrieve product data.

This service acts as the application's **product database API**, providing information such as:

* Product names
* Product descriptions
* Product prices
* Product images
* Product categories

Without the Product Catalog Service, users would not be able to browse products in the store.

---

# Product Catalog Service Architecture

```text
User
 |
 v
Frontend
 |
 v
Product Catalog Service
 |
 v
Product Information
```

Example:

```text
User Opens Store
       |
       v
Frontend Requests Products
       |
       v
Product Catalog Service
       |
       v
Returns Product List
```

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
* Automatic Pod recreation

---

# 2. Metadata

```yaml
metadata:
  name: opentelemetry-demo-productcatalogservice
```

## Purpose

Deployment name:

```text
opentelemetry-demo-productcatalogservice
```

Kubernetes uses this name to manage the Product Catalog Service Deployment.

---

# 3. Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-productcatalogservice
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: productcatalogservice
  app.kubernetes.io/name: opentelemetry-demo-productcatalogservice
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

## Purpose

Labels help Kubernetes:

* Identify resources
* Select Pods
* Organize components
* Enable monitoring
* Support service discovery

### Example

```yaml
app.kubernetes.io/component: productcatalogservice
```

Shows that this resource belongs to the Product Catalog Service.

---

# 4. Deployment Specification

```yaml
spec:
```

Defines how the Product Catalog Service should run.

---

## 4.1 Replicas

```yaml
replicas: 1
```

### Purpose

Runs one Product Catalog Service Pod.

```text
Product Catalog Deployment
            |
            v
 Product Catalog Pod
```

If the Pod crashes, Kubernetes automatically creates a replacement.

---

## 4.2 Revision History

```yaml
revisionHistoryLimit: 10
```

### Purpose

Stores the last 10 deployment versions.

This allows rollback if an update fails.

---

# 5. Selector

```yaml
selector:
  matchLabels:
    opentelemetry.io/name: opentelemetry-demo-productcatalogservice
```

## Purpose

The Deployment manages Pods that contain this label.

---

# 6. Pod Template

```yaml
template:
```

Defines how Product Catalog Pods should be created.

---

## Pod Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-productcatalogservice
```

Applied automatically to every Pod created by this Deployment.

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

This provides secure access to Kubernetes resources when required.

---

# 8. Product Catalog Container

```yaml
containers:
  - name: productcatalogservice
```

This is the main application container.

---

## 8.1 Container Image

```yaml
image: abhishekf5/product-catalog:13134113508
```

### Breakdown

```text
Registry/Image : abhishekf5/product-catalog
Tag            : 13134113508
```

### Purpose

Contains the Product Catalog Service application code.

### Note

Unlike most OpenTelemetry Demo services, this deployment uses a custom image:

```text
abhishekf5/product-catalog:13134113508
```

instead of:

```text
ghcr.io/open-telemetry/demo
```

This may contain customized product catalog functionality.

---

## 8.2 Image Pull Policy

```yaml
imagePullPolicy: IfNotPresent
```

### Purpose

* Uses local image if already available
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

The Product Catalog Service listens on:

```text
Port 8080
```

Other microservices access the service through this port.

---

# 10. OpenTelemetry Configuration

## Service Name

```yaml
OTEL_SERVICE_NAME
```

Resolved value:

```text
productcatalogservice
```

### Purpose

Identifies the service in:

* Traces
* Metrics
* Logs

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

Exports cumulative metrics.

---

# 11. Product Catalog Service Port

```yaml
PRODUCT_CATALOG_SERVICE_PORT=8080
```

## Purpose

Defines the port used by the Product Catalog Service.

```text
Product Catalog Service
           |
           v
         8080
```

This must match the container port.

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

Connects the Product Catalog Service to the Flagd feature flag system.

Feature flags can be used to:

* Enable experimental features
* Test application behavior
* Simulate scenarios for observability demonstrations

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

Data includes:

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
service.name=productcatalogservice,
service.namespace=opentelemetry-demo,
service.version=1.12.0
```

### Purpose

Adds metadata to telemetry.

Used in:

* Grafana
* Jaeger
* Prometheus
* OpenTelemetry Collector

---

# 15. Resource Limits

```yaml
resources:
  limits:
    memory: 20Mi
```

## Purpose

Limits memory usage to:

```text
20 MiB
```

### Why So Small?

The Product Catalog Service mainly:

* Reads product information
* Returns product data
* Performs lightweight operations

It does not perform heavy computations.

---

# 16. Volume Mounts

```yaml
volumeMounts:
```

No volume mounts are configured.

### Meaning

The application runs completely from the container image.

---

# 17. Volumes

```yaml
volumes:
```

No volumes are defined.

### Meaning

No persistent storage is required.

---

# Product Request Flow

```text
User Opens Store
        |
        v
Frontend
        |
        v
Product Catalog Service
        |
        +--> Product Names
        +--> Product Prices
        +--> Product Descriptions
        +--> Product Images
        |
        v
Frontend Displays Products
```

---

# Checkout Flow Dependency

```text
Frontend
    |
    v
Product Catalog Service
    |
    v
Product Selected
    |
    v
Cart Service
    |
    v
Checkout Service
```

Without the Product Catalog Service, users cannot select products.

---

# Observability Flow

```text
Product Catalog Service
          |
          v
OpenTelemetry Collector
          |
          +--> Jaeger
          +--> Prometheus
          +--> Grafana
```

Every product request generates telemetry.

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
Connected to Flagd
        |
        v
Connected to OTEL Collector
        |
        v
Ready to Serve Product Requests
```

---

# Summary

This Deployment:

* Deploys the Product Catalog Service.
* Uses image `abhishekf5/product-catalog:13134113508`.
* Listens on port `8080`.
* Provides product information to the store.
* Integrates with Flagd for feature flags.
* Sends telemetry to the OpenTelemetry Collector.
* Uses only `20Mi` of memory.
* Supports rolling updates and self-healing.

## In Simple Terms

The Product Catalog Service is the **product information provider** of the OpenTelemetry Demo store. Whenever users browse products, search items, or view product details, this service supplies the required product data. It also sends traces, metrics, and logs to the OpenTelemetry Collector so that product-related activity can be monitored through observability tools such as Jaeger, Grafana, and Prometheus.
