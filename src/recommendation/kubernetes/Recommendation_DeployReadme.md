# Recommendation Service Deployment (`deploy.yaml`) Documentation

## Overview

This Kubernetes Deployment file deploys the **Recommendation Service** in the OpenTelemetry Demo application.

The Recommendation Service is responsible for suggesting related or recommended products to users while they browse the online store. When a customer views a product, this 
service analyzes the request and returns recommended products that might also interest the customer.

For example:

```text
User Views Laptop
       |
       v
Recommendation Service
       |
       v
Suggests:
- Mouse
- Keyboard
- Laptop Bag
```

This service improves the shopping experience by providing personalized or related product recommendations.

---

# Recommendation Service Architecture

```text
User
 |
 v
Frontend
 |
 v
Recommendation Service
 |
 v
Product Catalog Service
 |
 v
Recommended Products
```

The Recommendation Service communicates with the Product Catalog Service to retrieve product information before generating recommendations.

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
* Rollback support

---

# 2. Metadata

```yaml
metadata:
  name: opentelemetry-demo-recommendationservice
```

## Purpose

Deployment name:

```text
opentelemetry-demo-recommendationservice
```

Kubernetes uses this name to manage the Recommendation Service deployment.

---

# 3. Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-recommendationservice
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: recommendationservice
  app.kubernetes.io/name: opentelemetry-demo-recommendationservice
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

## Purpose

Labels help Kubernetes:

* Identify resources
* Group components
* Enable monitoring
* Support service discovery
* Select Pods

### Example

```yaml
app.kubernetes.io/component: recommendationservice
```

Identifies this component as the Recommendation Service.

---

# 4. Deployment Specification

```yaml
spec:
```

Defines how the Recommendation Service should run.

---

## 4.1 Replicas

```yaml
replicas: 1
```

### Purpose

Runs one Recommendation Service Pod.

```text
Recommendation Deployment
            |
            v
Recommendation Pod
```

If the Pod crashes, Kubernetes automatically recreates it.

---

## 4.2 Revision History

```yaml
revisionHistoryLimit: 10
```

### Purpose

Stores the last 10 deployment versions.

Allows rollback if a deployment update causes issues.

---

# 5. Selector

```yaml
selector:
  matchLabels:
    opentelemetry.io/name: opentelemetry-demo-recommendationservice
```

## Purpose

The Deployment manages Pods that match this label.

---

# 6. Pod Template

```yaml
template:
```

Defines how Recommendation Service Pods should be created.

---

## Pod Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-recommendationservice
```

Automatically applied to all Pods.

---

# 7. Service Account

```yaml
serviceAccountName: opentelemetry-demo
```

## Purpose

The Pod runs using:

```text
opentelemetry-demo
```

Service Account.

Provides controlled access to Kubernetes resources.

---

# 8. Recommendation Service Container

```yaml
containers:
  - name: recommendationservice
```

Main application container.

---

## 8.1 Container Image

```yaml
image: ghcr.io/open-telemetry/demo:1.12.0-recommendationservice
```

### Breakdown

```text
Registry : ghcr.io
Project  : open-telemetry/demo
Version  : 1.12.0
Service  : recommendationservice
```

### Purpose

Contains the Recommendation Service application code.

---

## 8.2 Image Pull Policy

```yaml
imagePullPolicy: IfNotPresent
```

### Purpose

* Uses local image if available
* Downloads image only when required

### Benefit

Reduces startup time and bandwidth usage.

---

# 9. Container Port

```yaml
ports:
  - containerPort: 8080
```

## Purpose

The Recommendation Service listens on:

```text
Port 8080
```

Other services use this port to communicate.

---

# 10. OpenTelemetry Configuration

## Service Name

```yaml
OTEL_SERVICE_NAME
```

Resolved value:

```text
recommendationservice
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

# 11. Recommendation Service Port

```yaml
RECOMMENDATION_SERVICE_PORT=8080
```

## Purpose

Defines the application's listening port.

```text
Recommendation Service
          |
          v
        8080
```

---

# 12. Product Catalog Service Connection

```yaml
PRODUCT_CATALOG_SERVICE_ADDR=opentelemetry-demo-productcatalogservice:8080
```

## Purpose

Allows the Recommendation Service to communicate with the Product Catalog Service.

### Flow

```text
Recommendation Service
          |
          v
Product Catalog Service
          |
          v
Product Information
```

The Recommendation Service uses this information to generate recommendations.

### Example

```text
User Views Phone
         |
         v
Recommendation Service
         |
         v
Product Catalog Service
         |
         v
Fetch Product Data
         |
         v
Generate Recommendations
```

---

# 13. Python Log Correlation

```yaml
OTEL_PYTHON_LOG_CORRELATION=true
```

## Purpose

Links logs with traces.

### Benefit

When viewing a trace in Jaeger or Grafana:

```text
Trace
  |
  +--> Related Logs
```

This makes troubleshooting easier.

---

# 14. Python Protocol Buffer Configuration

```yaml
PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python
```

## Purpose

Forces Python Protocol Buffers to use the Python implementation.

This improves compatibility with telemetry instrumentation.

---

# 15. Feature Flag Configuration

## Flagd Host

```yaml
FLAGD_HOST=opentelemetry-demo-flagd
```

## Flagd Port

```yaml
FLAGD_PORT=8013
```

### Purpose

Connects the service to the Flagd feature flag system.

Used for:

* Feature testing
* Demo scenarios
* Controlled application behavior

---

# 16. Telemetry Export Endpoint

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

# 17. Resource Attributes

```yaml
OTEL_RESOURCE_ATTRIBUTES
```

Resolved value:

```text
service.name=recommendationservice,
service.namespace=opentelemetry-demo,
service.version=1.12.0
```

### Purpose

Adds service metadata to telemetry.

Used in:

* Jaeger
* Grafana
* Prometheus
* OpenTelemetry Collector

---

# 18. Resource Limits

```yaml
resources:
  limits:
    memory: 500Mi
```

## Purpose

Limits memory usage to:

```text
500 MiB
```

### Why Higher Memory?

Recommendation engines often:

* Process product relationships
* Analyze recommendations
* Store recommendation data in memory
* Handle multiple requests

Therefore more memory is allocated compared to other services.

---

# 19. Volume Mounts

```yaml
volumeMounts:
```

No volume mounts are configured.

### Meaning

The service runs entirely from the container image.

---

# 20. Volumes

```yaml
volumes:
```

No volumes are defined.

### Meaning

No persistent storage is required.

---

# Recommendation Flow

```text
User Views Product
         |
         v
Frontend
         |
         v
Recommendation Service
         |
         v
Product Catalog Service
         |
         v
Get Product Details
         |
         v
Generate Recommendations
         |
         v
Return Recommended Products
```

---

# Observability Flow

```text
Recommendation Service
          |
          v
OpenTelemetry Collector
          |
          +--> Jaeger
          +--> Prometheus
          +--> Grafana
```

All recommendation requests generate telemetry.

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
Connected to Product Catalog Service
        |
        v
Connected to Flagd
        |
        v
Connected to OTEL Collector
        |
        v
Ready to Generate Recommendations
```

---

# Summary

This Deployment:

* Deploys the Recommendation Service.
* Uses image `ghcr.io/open-telemetry/demo:1.12.0-recommendationservice`.
* Listens on port `8080`.
* Connects to the Product Catalog Service.
* Integrates with Flagd feature flags.
* Sends telemetry to the OpenTelemetry Collector.
* Enables Python log correlation.
* Uses `500Mi` memory.
* Supports self-healing and rolling updates.

## In Simple Terms

The Recommendation Service is the **product suggestion engine** of the OpenTelemetry Demo application. When a user views a product, this service communicates with the Product Catalog Service, analyzes product information, and returns recommended products that the user may also be interested in. It is fully integrated with OpenTelemetry, allowing all recommendation requests to be monitored through Jaeger, Grafana, Prometheus, and the OpenTelemetry Collector.
