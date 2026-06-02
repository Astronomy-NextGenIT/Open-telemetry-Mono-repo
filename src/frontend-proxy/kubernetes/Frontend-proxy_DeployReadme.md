# Frontend Proxy Deployment (`deploy.yaml`) Documentation

## Overview

This Kubernetes Deployment file is responsible for deploying the **Frontend Proxy Service** of the OpenTelemetry Demo application.

The Frontend Proxy acts as the **entry point** for users accessing the application. It receives incoming requests and routes them to the appropriate backend services such as the Frontend, Grafana, Jaeger, Image Provider, and Feature Flag services.

Think of it as a **traffic controller** that directs requests to the correct destination.

```text
User Browser
      |
      v
Frontend Proxy
      |
      +--> Frontend
      +--> Grafana
      +--> Jaeger
      +--> Flagd
      +--> Image Provider
```

---

# 1. API Version and Resource Type

```yaml
apiVersion: apps/v1
kind: Deployment
```

## Purpose

* `apps/v1` → Kubernetes Deployment API version.
* `Deployment` → Manages Pods automatically.

### Benefits

* Automatic pod creation
* Self-healing
* Rolling updates
* Rollbacks
* Scaling support

---

# 2. Metadata

```yaml
metadata:
  name: opentelemetry-demo-frontendproxy
```

## Purpose

This is the unique name of the Deployment.

```text
opentelemetry-demo-frontendproxy
```

Kubernetes uses this name to manage the Frontend Proxy Deployment.

---

# 3. Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-frontendproxy
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: frontendproxy
  app.kubernetes.io/name: opentelemetry-demo-frontendproxy
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

## Purpose of Labels

Labels help Kubernetes identify and organize resources.

### Example

```yaml
app.kubernetes.io/component: frontendproxy
```

This indicates that the Pod belongs to the Frontend Proxy component.

### Why Labels Matter

* Pod selection
* Monitoring
* Resource grouping
* Service discovery

---

# 4. Deployment Specification

```yaml
spec:
```

Defines how the Frontend Proxy should run.

---

## 4.1 Number of Replicas

```yaml
replicas: 1
```

### Purpose

Only one Frontend Proxy Pod will run.

```text
Frontend Proxy
      |
      └── Pod-1
```

If the Pod crashes, Kubernetes automatically creates a new one.

---

## 4.2 Revision History

```yaml
revisionHistoryLimit: 10
```

### Purpose

Stores the last 10 deployment versions.

### Benefit

Allows rollback if a deployment update causes issues.

---

# 5. Pod Selector

```yaml
selector:
  matchLabels:
    opentelemetry.io/name: opentelemetry-demo-frontendproxy
```

## Purpose

The Deployment manages Pods that contain this label.

Kubernetes uses this label to identify which Pods belong to this Deployment.

---

# 6. Pod Template

```yaml
template:
```

Defines how Kubernetes should create Frontend Proxy Pods.

---

## Pod Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-frontendproxy
  app.kubernetes.io/component: frontendproxy
```

These labels are automatically applied to every Pod created by this Deployment.

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

This provides secure access to Kubernetes resources when required.

---

# 8. Main Container

```yaml
containers:
  - name: frontendproxy
```

This is the main application container running the Frontend Proxy.

---

## 8.1 Container Image

```yaml
image: ghcr.io/open-telemetry/demo:1.12.0-frontendproxy
```

### Breakdown

```text
Registry : ghcr.io
Project  : open-telemetry/demo
Version  : 1.12.0
Service  : frontendproxy
```

### Purpose

This image contains the Frontend Proxy application.

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

Faster deployment and reduced network usage.

---

# 9. Container Port

```yaml
ports:
  - containerPort: 8080
    name: service
```

## Purpose

The Frontend Proxy listens for incoming requests on:

```text
Port 8080
```

### Flow

```text
User Request
      |
      v
Frontend Proxy
Port 8080
```

---

# 10. Environment Variables

Environment variables configure how the Frontend Proxy communicates with other services.

---

## 10.1 OTEL_SERVICE_NAME

```yaml
OTEL_SERVICE_NAME
```

Value comes from:

```yaml
metadata.labels['app.kubernetes.io/component']
```

Resolved value:

```text
frontendproxy
```

### Purpose

Used by OpenTelemetry to identify this service in traces and metrics.

---

## 10.2 OpenTelemetry Collector

```yaml
OTEL_COLLECTOR_NAME=opentelemetry-demo-otelcol
```

### Purpose

Specifies the OpenTelemetry Collector service.

---

## 10.3 Metrics Temporality

```yaml
OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE=cumulative
```

### Purpose

Metrics are sent as cumulative totals.

Example:

```text
Minute 1 = 100 requests
Minute 2 = 200 requests
Minute 3 = 300 requests
```

---

## 10.4 Envoy Port

```yaml
ENVOY_PORT=8080
```

### Purpose

The proxy listens on port 8080.

---

# 11. Feature Flag Service Configuration

## Flagd Host

```yaml
FLAGD_HOST=opentelemetry-demo-flagd
```

## Flagd Port

```yaml
FLAGD_PORT=8013
```

### Purpose

Frontend Proxy communicates with Flagd to retrieve feature flags.

Feature flags allow enabling or disabling application features without redeploying the application.

---

# 12. Feature Flag UI Configuration

```yaml
FLAGD_UI_HOST=opentelemetry-demo-flagd
FLAGD_UI_PORT=4000
```

### Purpose

Provides access to the Flagd web interface.

---

# 13. Frontend Service Configuration

```yaml
FRONTEND_HOST=opentelemetry-demo-frontend
FRONTEND_PORT=8080
```

### Purpose

Specifies where the actual Frontend application is running.

### Flow

```text
User
  |
Frontend Proxy
  |
Frontend Service
```

---

# 14. Grafana Configuration

```yaml
GRAFANA_SERVICE_HOST=opentelemetry-demo-grafana
GRAFANA_SERVICE_PORT=80
```

### Purpose

Allows access to Grafana dashboards through the proxy.

Grafana is used for:

* Monitoring
* Dashboards
* Metrics visualization

---

# 15. Image Provider Configuration

```yaml
IMAGE_PROVIDER_HOST=opentelemetry-demo-imageprovider
IMAGE_PROVIDER_PORT=8081
```

### Purpose

Provides product images used by the frontend application.

---

# 16. Jaeger Configuration

```yaml
JAEGER_SERVICE_HOST=opentelemetry-demo-jaeger-query
JAEGER_SERVICE_PORT=16686
```

### Purpose

Allows access to Jaeger tracing UI.

Jaeger helps developers:

* View distributed traces
* Troubleshoot requests
* Analyze service dependencies

---

# 17. Load Generator Configuration

```yaml
LOCUST_WEB_HOST=opentelemetry-demo-loadgenerator
LOCUST_WEB_PORT=8089
```

### Purpose

Provides access to the Locust load-testing dashboard.

Used for:

* Performance testing
* Traffic simulation
* Stress testing

---

# 18. OpenTelemetry Collector Configuration

```yaml
OTEL_COLLECTOR_HOST=$(OTEL_COLLECTOR_NAME)
```

Resolves to:

```text
opentelemetry-demo-otelcol
```

### Collector Ports

```yaml
OTEL_COLLECTOR_PORT_GRPC=4317
OTEL_COLLECTOR_PORT_HTTP=4318
```

### Purpose

Telemetry data is sent to the OpenTelemetry Collector using:

* gRPC → 4317
* HTTP → 4318

---

# 19. Resource Attributes

```yaml
OTEL_RESOURCE_ATTRIBUTES
```

Resolved value:

```text
service.name=frontendproxy,
service.namespace=opentelemetry-demo,
service.version=1.12.0
```

### Purpose

Adds metadata to traces, metrics, and logs.

This helps monitoring tools identify:

* Service name
* Namespace
* Version

---

# 20. Resource Limits

```yaml
resources:
  limits:
    memory: 50Mi
```

## Purpose

Maximum memory allowed:

```text
50 MiB
```

### Why Important

Prevents the container from consuming excessive memory.

---

# 21. Security Context

```yaml
securityContext:
  runAsGroup: 101
  runAsNonRoot: true
  runAsUser: 101
```

## Purpose

Improves container security.

### runAsNonRoot

```yaml
runAsNonRoot: true
```

Ensures the container does not run as the root user.

### runAsUser

```yaml
runAsUser: 101
```

Runs the process using User ID 101.

### runAsGroup

```yaml
runAsGroup: 101
```

Runs the process using Group ID 101.

### Benefit

Reduces security risks if the container is compromised.

---

# 22. Volume Mounts

```yaml
volumeMounts:
```

No volume mounts are configured.

### Meaning

The service does not require persistent storage.

---

# 23. Volumes

```yaml
volumes:
```

No volumes are defined.

### Meaning

The application runs entirely from the container image.

---

# Request Flow Through Frontend Proxy

```text
User Browser
      |
      v
Frontend Proxy (8080)
      |
      +------------------+
      |                  |
      v                  v
Frontend             Flagd
      |
      +------------------+
      |                  |
      v                  v
Grafana             Jaeger
      |
      v
Image Provider
```

---

# Deployment Lifecycle

```text
Deployment Created
        |
        v
Create Frontend Proxy Pod
        |
        v
Start Container
        |
        v
Listen on Port 8080
        |
        v
Connect to:
  - Frontend
  - Grafana
  - Jaeger
  - Flagd
  - Image Provider
  - Load Generator
        |
        v
Send Telemetry Data
to OTEL Collector
        |
        v
Serve User Requests
```

# Summary

This Deployment:

* Creates 1 Frontend Proxy Pod.
* Uses image `ghcr.io/open-telemetry/demo:1.12.0-frontendproxy`.
* Listens on port `8080`.
* Acts as the entry point for users.
* Routes requests to Frontend, Grafana, Jaeger, Flagd, and Image Provider services.
* Sends telemetry data to the OpenTelemetry Collector.
* Uses secure non-root execution (`UID 101`).
* Limits memory usage to `50Mi`.
* Supports Kubernetes self-healing, updates, and rollbacks.

**In simple terms:** The Frontend Proxy is the **gateway of the OpenTelemetry Demo application**. It receives user requests, forwards them to the correct backend services, exposes observability tools like Grafana and Jaeger, and sends telemetry data to the OpenTelemetry monitoring stack.
