# Ad Service Deployment Documentation

## Overview

This `deploy.yaml` file is responsible for deploying the **Ad Service** in the Kubernetes cluster.

The Ad Service is a microservice that serves advertisement-related data to other components of the OpenTelemetry Demo application. This Deployment ensures that the Ad Service container is running, monitored, and automatically recreated if it fails.

Unlike the Accounting Service, this service exposes an application port (`8080`) because it receives requests from other services within the application.

---

# High-Level Deployment Flow

```text
deploy.yaml
      ↓
Deployment Created
      ↓
ReplicaSet Created
      ↓
Pod Created
      ↓
Ad Service Container Started
      ↓
Port 8080 Exposed Inside Pod
      ↓
Connects To Flagd Feature Flag Service
      ↓
Telemetry Sent To OpenTelemetry Collector
      ↓
Ad Service Ready
```

---

# Resource Type

```yaml
apiVersion: apps/v1
kind: Deployment
```

## Purpose

This tells Kubernetes:

* Use Deployment API from `apps/v1`
* Create a Deployment resource

### What Happens

When executed:

```bash
kubectl apply -f deploy.yaml
```

Kubernetes creates a Deployment that manages the Ad Service lifecycle.

---

# Deployment Name

```yaml
metadata:
  name: opentelemetry-demo-adservice
```

## Purpose

Provides a unique name for this Deployment.

### Result

Kubernetes manages the Ad Service using this identifier.

---

# Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-adservice
  app.kubernetes.io/component: adservice
  app.kubernetes.io/name: opentelemetry-demo-adservice
  app.kubernetes.io/version: "1.12.0"
```

## Purpose

Labels act as metadata.

### Benefits

They help Kubernetes:

* Identify resources
* Group related resources
* Connect Services to Pods
* Enable monitoring and filtering

Example:

```yaml
app.kubernetes.io/component: adservice
```

indicates that this resource belongs to the Ad Service component.

---

# Replica Configuration

```yaml
replicas: 1
```

## Purpose

Defines the number of Pod instances.

### What Happens

Kubernetes ensures:

```text
1 Ad Service Pod
```

is always running.

If the Pod crashes:

```text
Pod Failure
     ↓
Deployment Detects Failure
     ↓
New Pod Created Automatically
```

---

# Revision History

```yaml
revisionHistoryLimit: 10
```

## Purpose

Stores previous Deployment versions.

### Benefit

Allows rollback during failed deployments.

```text
Version 1.12.0
      ↓
Version 1.13.0 Fails
      ↓
Rollback To Previous Version
```

---

# Selector

```yaml
selector:
  matchLabels:
    opentelemetry.io/name: opentelemetry-demo-adservice
```

## Purpose

Defines which Pods belong to this Deployment.

### What Happens

Deployment continuously checks for Pods containing this label and manages them.

---

# Pod Template

```yaml
template:
```

## Purpose

Acts as a blueprint for Pod creation.

### Flow

```text
Deployment
      ↓
Pod Template
      ↓
Pod Created
```

Every new Pod is created using this template.

---

# Service Account

```yaml
serviceAccountName: opentelemetry-demo
```

## Purpose

Associates the Pod with a Kubernetes Service Account.

### Benefit

Provides controlled permissions for Kubernetes resource access.

---

# Main Application Container

```yaml
containers:
  - name: adservice
```

## Purpose

Defines the main application container.

This container runs the Ad Service application.

---

# Container Image

```yaml
image: ghcr.io/open-telemetry/demo:1.12.0-adservice
```

## Purpose

Specifies the Docker image used to create the container.

### What Happens

Kubernetes pulls the image from:

```text
GitHub Container Registry (GHCR)
```

and starts the Ad Service application.

---

# Image Pull Policy

```yaml
imagePullPolicy: IfNotPresent
```

## Purpose

Controls image download behavior.

### Flow

```text
Image Exists On Node
        ↓
Use Existing Image

Image Missing
        ↓
Download Image
```

This speeds up container startup.

---

# Container Port

```yaml
ports:
  - containerPort: 8080
    name: service
```

## Purpose

Exposes port `8080` inside the container.

### What Happens

The Ad Service listens for incoming requests on:

```text
Port 8080
```

### Request Flow

```text
Other Microservice
        ↓
Ad Service Pod
        ↓
Port 8080
        ↓
Ad Service Application
```

---

# Environment Variables

Environment variables provide runtime configuration to the application.

---

## OTEL_SERVICE_NAME

```yaml
OTEL_SERVICE_NAME
```

### Purpose

Automatically obtains the service name from Pod labels.

### Result

```text
adservice
```

This name appears in traces, metrics, and logs.

---

## OTEL_COLLECTOR_NAME

```yaml
OTEL_COLLECTOR_NAME=opentelemetry-demo-otelcol
```

### Purpose

Specifies the OpenTelemetry Collector service.

### What Happens

Telemetry data is sent to this collector.

---

## Metrics Temporality

```yaml
OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE=cumulative
```

### Purpose

Determines how metrics are reported.

### Meaning

Metrics continuously accumulate values over time.

---

## AD_SERVICE_PORT

```yaml
AD_SERVICE_PORT=8080
```

### Purpose

Defines the application listening port.

### Result

The Ad Service runs on:

```text
Port 8080
```

---

## FLAGD_HOST

```yaml
FLAGD_HOST=opentelemetry-demo-flagd
```

## Purpose

Specifies the hostname of the Flagd service.

### What Is Flagd?

Flagd is a feature flag management service.

It allows enabling or disabling application features dynamically without redeploying the application.

---

## FLAGD_PORT

```yaml
FLAGD_PORT=8013
```

### Purpose

Specifies the port used to communicate with Flagd.

### Connection

```text
Ad Service
      ↓
opentelemetry-demo-flagd
      ↓
Port 8013
```

---

## OpenTelemetry Endpoint

```yaml
OTEL_EXPORTER_OTLP_ENDPOINT=http://$(OTEL_COLLECTOR_NAME):4318
```

### Final Value

```text
http://opentelemetry-demo-otelcol:4318
```

### Purpose

Defines where telemetry data should be exported.

### Data Sent

* Traces
* Metrics
* Logs

---

## OTEL_LOGS_EXPORTER

```yaml
OTEL_LOGS_EXPORTER=otlp
```

### Purpose

Configures log exporting.

### What Happens

Application logs are forwarded to the OpenTelemetry Collector.

---

## Resource Attributes

```yaml
OTEL_RESOURCE_ATTRIBUTES
```

### Purpose

Adds metadata to telemetry data.

### Example

```text
service.name=adservice
service.namespace=opentelemetry-demo
service.version=1.12.0
```

This helps identify the service in observability dashboards.

---

# Resource Limits

```yaml
resources:
  limits:
    memory: 300Mi
```

## Purpose

Restricts memory usage.

### Maximum Allowed Memory

```text
300 MiB
```

### Benefit

Prevents excessive resource consumption.

---

# Volumes and Volume Mounts

```yaml
volumeMounts:
volumes:
```

## Purpose

Provide storage inside containers.

### Current State

No volumes are defined in this Deployment.

The sections are present for future storage requirements.

---

# Complete Runtime Flow

```text
kubectl apply deploy.yaml
            ↓
Deployment Created
            ↓
ReplicaSet Created
            ↓
Pod Created
            ↓
Ad Service Container Started
            ↓
Port 8080 Opened
            ↓
Feature Flag Configuration Loaded
            ↓
Connects To Flagd Service
            ↓
Telemetry Configuration Loaded
            ↓
Connects To OpenTelemetry Collector
            ↓
Metrics, Logs, And Traces Exported
            ↓
Ad Service Ready
```

---

# Service Communication Flow

```text
Frontend Service
        ↓
Ad Service
        ↓
Port 8080
        ↓
Business Logic Executed
        ↓
Feature Flags Retrieved From Flagd
        ↓
Response Returned
```

---

# Summary

This Deployment file:

* Deploys the Ad Service application
* Maintains one running Pod
* Uses a Docker image from GitHub Container Registry
* Exposes application port 8080
* Integrates with Flagd for feature flags
* Sends metrics, traces, and logs to OpenTelemetry Collector
* Automatically recreates failed Pods
* Supports rollback through Deployment history
* Restricts memory usage to 300Mi

The overall goal of this Deployment is to ensure that the Ad Service remains available, communicates with Flagd for feature management, and continuously sends observability data to the OpenTelemetry platform.
