# Frontend Deployment (`deploy.yaml`) Documentation

## Overview

This Kubernetes Deployment file deploys the **Frontend Service** of the OpenTelemetry Demo application.

The Frontend Service is the **main web application** that users interact with. It receives requests from the Frontend Proxy and communicates with multiple backend 
microservices such as Product Catalog, Cart, Checkout, Recommendation, Currency, Shipping, and Advertisement services.

```text
User
  |
  v
Frontend Proxy
  |
  v
Frontend Service
  |
  +--> Product Catalog Service
  +--> Cart Service
  +--> Checkout Service
  +--> Recommendation Service
  +--> Shipping Service
  +--> Currency Service
  +--> Ad Service
```

The Frontend Service also generates telemetry data (logs, metrics, and traces) and sends it to the OpenTelemetry Collector for monitoring.

---

# 1. API Version and Resource Type

```yaml
apiVersion: apps/v1
kind: Deployment
```

## Purpose

* `apps/v1` → Kubernetes Deployment API.
* `Deployment` → Manages application Pods.

### Benefits

* Automatic Pod creation
* Self-healing
* Rolling updates
* Rollbacks
* Scaling support

---

# 2. Metadata

```yaml
metadata:
  name: opentelemetry-demo-frontend
```

## Purpose

The deployment name is:

```text
opentelemetry-demo-frontend
```

This uniquely identifies the Frontend Deployment within the cluster.

---

# 3. Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-frontend
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: frontend
  app.kubernetes.io/name: opentelemetry-demo-frontend
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

## Purpose

Labels help Kubernetes:

* Identify resources
* Select Pods
* Group components
* Enable monitoring and service discovery

Example:

```yaml
app.kubernetes.io/component: frontend
```

Indicates that this resource belongs to the Frontend component.

---

# 4. Deployment Specification

```yaml
spec:
```

Defines how Kubernetes should run the Frontend application.

---

## 4.1 Number of Replicas

```yaml
replicas: 1
```

### Purpose

Runs one Frontend Pod.

```text
Frontend Deployment
         |
         v
      Frontend Pod
```

If the Pod fails, Kubernetes automatically creates a replacement.

---

## 4.2 Revision History

```yaml
revisionHistoryLimit: 10
```

### Purpose

Stores the last 10 deployment revisions.

### Benefit

Allows rollback if a deployment update causes issues.

---

# 5. Pod Selector

```yaml
selector:
  matchLabels:
    opentelemetry.io/name: opentelemetry-demo-frontend
```

## Purpose

The Deployment manages Pods with this label.

Kubernetes uses this label to identify which Pods belong to this Deployment.

---

# 6. Pod Template

```yaml
template:
```

Defines how Frontend Pods should be created.

---

## Pod Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-frontend
  app.kubernetes.io/component: frontend
```

These labels are automatically applied to every Frontend Pod.

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

# 8. Main Container

```yaml
containers:
  - name: frontend
```

This is the main application container running the Frontend Service.

---

## 8.1 Container Image

```yaml
image: ghcr.io/open-telemetry/demo:1.12.0-frontend
```

### Breakdown

```text
Registry : ghcr.io
Project  : open-telemetry/demo
Version  : 1.12.0
Service  : frontend
```

### Purpose

Contains the Frontend application code.

---

## 8.2 Image Pull Policy

```yaml
imagePullPolicy: IfNotPresent
```

### Purpose

Kubernetes:

* Uses local image if available
* Downloads image only if necessary

### Benefit

Faster startup and reduced network usage.

---

# 9. Container Port

```yaml
ports:
  - containerPort: 8080
```

## Purpose

The Frontend application listens on:

```text
Port 8080
```

### Flow

```text
Frontend Proxy
      |
      v
Frontend Service
Port 8080
```

---

# 10. OpenTelemetry Configuration

## Service Name

```yaml
OTEL_SERVICE_NAME
```

Resolved value:

```text
frontend
```

### Purpose

Identifies the Frontend service in telemetry data.

---

## Collector Name

```yaml
OTEL_COLLECTOR_NAME=opentelemetry-demo-otelcol
```

### Purpose

Specifies the OpenTelemetry Collector service.

---

## Metrics Temporality

```yaml
OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE=cumulative
```

### Purpose

Metrics are sent as cumulative values.

---

# 11. Frontend Application Configuration

## Frontend Port

```yaml
FRONTEND_PORT=8080
```

### Purpose

Defines the application listening port.

---

## Frontend Address

```yaml
FRONTEND_ADDR=:8080
```

### Purpose

Tells the application to listen on all network interfaces using port 8080.

```text
0.0.0.0:8080
```

---

# 12. Backend Service Connections

The Frontend communicates with multiple backend microservices.

---

## Advertisement Service

```yaml
AD_SERVICE_ADDR=opentelemetry-demo-adservice:8080
```

### Purpose

Retrieves advertisements displayed on the website.

---

## Cart Service

```yaml
CART_SERVICE_ADDR=opentelemetry-demo-cartservice:8080
```

### Purpose

Handles shopping cart operations.

---

## Checkout Service

```yaml
CHECKOUT_SERVICE_ADDR=opentelemetry-demo-checkoutservice:8080
```

### Purpose

Processes customer orders and payments.

---

## Currency Service

```yaml
CURRENCY_SERVICE_ADDR=opentelemetry-demo-currencyservice:8080
```

### Purpose

Converts prices between currencies.

---

## Product Catalog Service

```yaml
PRODUCT_CATALOG_SERVICE_ADDR=opentelemetry-demo-productcatalogservice:8080
```

### Purpose

Provides product information displayed on the website.

---

## Recommendation Service

```yaml
RECOMMENDATION_SERVICE_ADDR=opentelemetry-demo-recommendationservice:8080
```

### Purpose

Suggests related products to customers.

---

## Shipping Service

```yaml
SHIPPING_SERVICE_ADDR=opentelemetry-demo-shippingservice:8080
```

### Purpose

Calculates shipping costs and delivery details.

---

# 13. Feature Flag Service

## Host

```yaml
FLAGD_HOST=opentelemetry-demo-flagd
```

## Port

```yaml
FLAGD_PORT=8013
```

### Purpose

Provides feature flag configuration.

Feature flags allow application features to be enabled or disabled without redeployment.

---

# 14. OpenTelemetry Collector Endpoint

## Collector Host

```yaml
OTEL_COLLECTOR_HOST=$(OTEL_COLLECTOR_NAME)
```

Resolves to:

```text
opentelemetry-demo-otelcol
```

---

## OTLP Endpoint

```yaml
OTEL_EXPORTER_OTLP_ENDPOINT=http://opentelemetry-demo-otelcol:4317
```

### Purpose

Sends telemetry data to the OpenTelemetry Collector using gRPC.

---

# 15. Browser Telemetry Configuration

## Web Telemetry Service Name

```yaml
WEB_OTEL_SERVICE_NAME=frontend-web
```

### Purpose

Identifies telemetry generated from the user's browser.

Example:

```text
frontend-web
```

Appears separately in tracing tools.

---

## Browser Trace Endpoint

```yaml
PUBLIC_OTEL_EXPORTER_OTLP_TRACES_ENDPOINT=http://localhost:8080/otlp-http/v1/traces
```

### Purpose

Frontend JavaScript sends browser traces to this endpoint.

Flow:

```text
User Browser
      |
      v
Frontend Service
      |
      v
OpenTelemetry Collector
```

This enables end-to-end tracing from browser to backend services.

---

# 16. Resource Attributes

```yaml
OTEL_RESOURCE_ATTRIBUTES
```

Resolved value:

```text
service.name=frontend,
service.namespace=opentelemetry-demo,
service.version=1.12.0
```

### Purpose

Adds metadata to logs, traces, and metrics.

Helps monitoring tools identify:

* Service name
* Namespace
* Version

---

# 17. Resource Limits

```yaml
resources:
  limits:
    memory: 250Mi
```

## Purpose

Maximum memory allowed:

```text
250 MiB
```

### Benefit

Prevents excessive memory consumption.

---

# 18. Security Context

```yaml
securityContext:
  runAsGroup: 1001
  runAsNonRoot: true
  runAsUser: 1001
```

## Purpose

Improves container security.

### runAsNonRoot

```yaml
runAsNonRoot: true
```

Prevents the container from running as root.

### runAsUser

```yaml
runAsUser: 1001
```

Runs the application using User ID 1001.

### runAsGroup

```yaml
runAsGroup: 1001
```

Runs the application using Group ID 1001.

### Benefit

Reduces security risks if the container is compromised.

---

# 19. Volume Mounts

```yaml
volumeMounts:
```

No volume mounts are configured.

### Meaning

The application does not require persistent storage.

---

# 20. Volumes

```yaml
volumes:
```

No volumes are defined.

### Meaning

The application runs entirely from the container image.

---

# Frontend Service Architecture

```text
User Browser
      |
      v
Frontend Proxy
      |
      v
Frontend Service
      |
      +--> Ad Service
      +--> Product Catalog Service
      +--> Cart Service
      +--> Checkout Service
      +--> Recommendation Service
      +--> Currency Service
      +--> Shipping Service
      |
      v
OpenTelemetry Collector
```

---

# Deployment Flow

```text
Deployment Created
        |
        v
Create Frontend Pod
        |
        v
Start Frontend Container
        |
        v
Listen on Port 8080
        |
        v
Connect to Backend Services
        |
        v
Receive Requests from Frontend Proxy
        |
        v
Generate Telemetry Data
        |
        v
Send Traces, Logs, and Metrics
to OpenTelemetry Collector
```

# Summary

This Deployment:

* Creates one Frontend Pod.
* Uses image `ghcr.io/open-telemetry/demo:1.12.0-frontend`.
* Listens on port `8080`.
* Communicates with Ad, Cart, Checkout, Currency, Product Catalog, Recommendation, and Shipping services.
* Uses Flagd for feature flags.
* Sends telemetry data to the OpenTelemetry Collector.
* Captures browser-side telemetry using `frontend-web`.
* Runs as a non-root user for security.
* Limits memory usage to `250Mi`.

## In Simple Terms

The Frontend Deployment hosts the **main shopping website** of the OpenTelemetry Demo application. It receives requests from the Frontend Proxy, fetches data from multiple backend microservices, displays information to users, and sends monitoring data to the OpenTelemetry observability stack.
