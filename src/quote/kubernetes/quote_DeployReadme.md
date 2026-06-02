# Quote Service Deployment (`deploy.yaml`) Documentation

## Overview

This Kubernetes Deployment file deploys the **Quote Service** in the OpenTelemetry Demo application.

The Quote Service is responsible for providing product-related quotes or pricing information that can be used by other services in the application. It acts as a backend microservice that responds to requests and returns quote data.

In the OpenTelemetry Demo, the Quote Service is mainly included to demonstrate microservice communication and observability features such as distributed tracing, metrics collection, and logging.

---

# Quote Service Architecture

```text
Frontend
    |
    v
Quote Service
    |
    v
Quote Information
```

Example Flow:

```text
Application Request
         |
         v
Quote Service
         |
         v
Returns Quote Data
```

Every request processed by the Quote Service generates telemetry data that can be viewed in observability tools.

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
  name: opentelemetry-demo-quoteservice
```

## Purpose

Deployment name:

```text
opentelemetry-demo-quoteservice
```

Kubernetes uses this name to identify and manage the deployment.

---

# 3. Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-quoteservice
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: quoteservice
  app.kubernetes.io/name: opentelemetry-demo-quoteservice
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

## Purpose

Labels help Kubernetes:

* Identify resources
* Group related components
* Select Pods
* Enable service discovery
* Support monitoring

### Example

```yaml
app.kubernetes.io/component: quoteservice
```

Shows that the resource belongs to the Quote Service.

---

# 4. Deployment Specification

```yaml
spec:
```

Defines how the Quote Service should run.

---

## 4.1 Replicas

```yaml
replicas: 1
```

### Purpose

Runs one Quote Service Pod.

```text
Quote Service Deployment
           |
           v
     Quote Service Pod
```

If the Pod fails, Kubernetes automatically recreates it.

---

## 4.2 Revision History

```yaml
revisionHistoryLimit: 10
```

### Purpose

Stores the last 10 deployment versions.

Allows rollback if a deployment update fails.

---

# 5. Selector

```yaml
selector:
  matchLabels:
    opentelemetry.io/name: opentelemetry-demo-quoteservice
```

## Purpose

The Deployment manages Pods matching this label.

---

# 6. Pod Template

```yaml
template:
```

Defines how Quote Service Pods should be created.

---

## Pod Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-quoteservice
```

Applied automatically to all Pods created by this Deployment.

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

Provides controlled access to Kubernetes resources if needed.

---

# 8. Quote Service Container

```yaml
containers:
  - name: quoteservice
```

Main application container running the Quote Service.

---

## 8.1 Container Image

```yaml
image: ghcr.io/open-telemetry/demo:1.12.0-quoteservice
```

### Breakdown

```text
Registry : ghcr.io
Project  : open-telemetry/demo
Version  : 1.12.0
Service  : quoteservice
```

### Purpose

Contains the Quote Service application code.

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

The Quote Service listens on:

```text
Port 8080
```

Other services communicate with it through this port.

---

# 10. OpenTelemetry Configuration

## Service Name

```yaml
OTEL_SERVICE_NAME
```

Resolved value:

```text
quoteservice
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

Specifies the OpenTelemetry Collector that receives telemetry data.

---

## Metrics Temporality

```yaml
OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE=cumulative
```

### Purpose

Exports metrics as cumulative values.

---

# 11. Quote Service Port Configuration

```yaml
QUOTE_SERVICE_PORT=8080
```

## Purpose

Defines the application listening port.

```text
Quote Service
      |
      v
    8080
```

Must match the container port configuration.

---

# 12. PHP Auto Instrumentation

```yaml
OTEL_PHP_AUTOLOAD_ENABLED=true
```

## Purpose

Enables automatic OpenTelemetry instrumentation for PHP applications.

### Benefit

Automatically captures:

* HTTP requests
* Response times
* Errors
* Traces

Without requiring manual instrumentation code.

---

# 13. Telemetry Export Endpoint

```yaml
OTEL_EXPORTER_OTLP_ENDPOINT=http://$(OTEL_COLLECTOR_NAME):4318
```

Resolved value:

```text
http://opentelemetry-demo-otelcol:4318
```

### Purpose

Sends telemetry data to the OpenTelemetry Collector.

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
service.name=quoteservice,
service.namespace=opentelemetry-demo,
service.version=1.12.0
```

### Purpose

Adds metadata to telemetry.

This information appears in:

* Grafana
* Jaeger
* Prometheus
* OpenTelemetry Collector

---

# 15. Resource Limits

```yaml
resources:
  limits:
    memory: 40Mi
```

## Purpose

Limits memory usage to:

```text
40 MiB
```

### Benefit

Prevents excessive memory consumption.

The Quote Service performs lightweight operations, so a small memory limit is sufficient.

---

# 16. Security Context

```yaml
securityContext:
  runAsGroup: 33
  runAsNonRoot: true
  runAsUser: 33
```

## Purpose

Runs the application securely.

### runAsNonRoot

```yaml
runAsNonRoot: true
```

Prevents the container from running as the root user.

---

### runAsUser

```yaml
runAsUser: 33
```

Runs the application as Linux User ID 33.

---

### runAsGroup

```yaml
runAsGroup: 33
```

Runs the application as Linux Group ID 33.

---

### Benefit

Improves container security by following the principle of least privilege.

---

# 17. Volume Mounts

```yaml
volumeMounts:
```

No volume mounts are configured.

### Meaning

The application runs entirely from the container image.

---

# 18. Volumes

```yaml
volumes:
```

No volumes are defined.

### Meaning

The service does not require persistent storage.

---

# Request Processing Flow

```text
Application Request
         |
         v
Quote Service
         |
         +--> Process Request
         |
         +--> Generate Response
         |
         v
Return Quote Data
```

---

# Observability Flow

```text
Quote Service
       |
       v
OpenTelemetry Collector
       |
       +--> Jaeger (Traces)
       +--> Prometheus (Metrics)
       +--> Grafana (Dashboards)
```

Every request handled by the Quote Service generates telemetry data.

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
Auto Instrumentation Enabled
        |
        v
Connected to OTEL Collector
        |
        v
Ready to Serve Requests
```

---

# Summary

This Deployment:

* Deploys the Quote Service.
* Uses image `ghcr.io/open-telemetry/demo:1.12.0-quoteservice`.
* Listens on port `8080`.
* Enables PHP auto-instrumentation.
* Sends telemetry to the OpenTelemetry Collector.
* Uses `40Mi` memory.
* Runs as a non-root user (`UID 33`).
* Supports self-healing and rolling updates.

## In Simple Terms

The Quote Service is a lightweight backend microservice that processes quote-related requests and returns quote information to other parts of the application. It is fully instrumented with OpenTelemetry, allowing every request to generate traces, metrics, and logs that can be viewed in Grafana, Jaeger, and Prometheus for observability and monitoring purposes.
