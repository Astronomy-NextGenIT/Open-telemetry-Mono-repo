# Email Service Deployment Documentation

## Overview

This `deploy.yaml` file is responsible for deploying the **Email Service** in the Kubernetes cluster.

The Email Service handles email-related operations within the OpenTelemetry Demo application. Whenever a customer successfully places an order, 
the Checkout Service communicates with the Email Service to send order confirmation notifications.

This Deployment ensures the Email Service is always available, monitored, restarted automatically if it fails, and integrated with OpenTelemetry for distributed tracing 
and observability.

---

## High-Level Architecture

```text
Customer Places Order
          ↓
Checkout Service
          ↓
Email Service
          ↓
Generate Confirmation Email
          ↓
Send Email Notification
```

---

## Deployment Resource

```yaml
apiVersion: apps/v1
kind: Deployment
```

### Purpose

Creates and manages Email Service Pods.

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
Email Service Started
```

---

## Deployment Name

```yaml
metadata:
  name: opentelemetry-demo-emailservice
```

### Purpose

Provides a unique identifier for the Email Service Deployment.

---

## Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-emailservice
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: emailservice
  app.kubernetes.io/name: opentelemetry-demo-emailservice
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

### Purpose

Labels help Kubernetes:

* Identify resources
* Group application components
* Support Service discovery
* Enable monitoring
* Simplify troubleshooting

---

## Replica Configuration

```yaml
replicas: 1
```

### Purpose

Ensures one Email Service Pod is always running.

### Failure Recovery

```text
Pod Crashes
      ↓
Deployment Detects Failure
      ↓
New Pod Created Automatically
```

---

## Revision History

```yaml
revisionHistoryLimit: 10
```

### Purpose

Stores the previous 10 Deployment versions.

### Benefit

Supports rollback during failed updates.

---

## Selector

```yaml
selector:
  matchLabels:
    opentelemetry.io/name: opentelemetry-demo-emailservice
```

### Purpose

Defines which Pods belong to this Deployment.

### Flow

```text
Deployment
      ↓
Find Matching Pods
      ↓
Manage Those Pods
```

---

## Pod Template

```yaml
template:
```

### Purpose

Acts as the blueprint used to create Email Service Pods.

---

## Service Account

```yaml
serviceAccountName: opentelemetry-demo
```

### Purpose

Associates the Pod with a Kubernetes Service Account.

### Benefit

Provides Kubernetes permissions required by the application.

---

## Main Application Container

```yaml
containers:
  - name: emailservice
```

### Purpose

Runs the Email Service application.

---

## Container Image

```yaml
image: ghcr.io/open-telemetry/demo:1.12.0-emailservice
```

### Purpose

Specifies the Docker image used to run the Email Service.

### Startup Flow

```text
GitHub Container Registry
            ↓
Image Pulled
            ↓
Container Created
            ↓
Email Service Started
```

---

## Image Pull Policy

```yaml
imagePullPolicy: IfNotPresent
```

### Purpose

Avoids downloading the image if it already exists on the node.

### Behavior

```text
Image Available Locally
          ↓
Use Existing Image

Image Missing
          ↓
Download Image
```

---

## Container Port

```yaml
ports:
  - containerPort: 8080
```

### Purpose

Exposes the Email Service application on port 8080.

### Result

```text
Email Service
      ↓
Port 8080
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

Automatically retrieves the service name from Kubernetes labels.

### Result

```text
emailservice
```

Used in traces, metrics, and logs.

---

## OTEL_COLLECTOR_NAME

```yaml
OTEL_COLLECTOR_NAME=opentelemetry-demo-otelcol
```

### Purpose

Defines the OpenTelemetry Collector service name.

---

## Metrics Temporality

```yaml
OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE=cumulative
```

### Purpose

Controls metric aggregation behavior.

### Meaning

```text
Metrics continuously accumulate over time.
```

---

## Email Service Port

```yaml
EMAIL_SERVICE_PORT=8080
```

### Purpose

Defines the listening port of the Email Service.

---

## Application Environment

```yaml
APP_ENV=production
```

### Purpose

Defines the application environment.

### Current Value

```text
production
```

### Benefits

* Production configuration enabled
* Optimized runtime settings
* Production logging behavior

---

## OpenTelemetry Trace Endpoint

```yaml
OTEL_EXPORTER_OTLP_TRACES_ENDPOINT=http://opentelemetry-demo-otelcol:4318/v1/traces
```

### Purpose

Specifies where trace data is exported.

### Data Sent

```text
Distributed Traces
```

### Flow

```text
Email Service
      ↓
Generate Trace
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

### Purpose

Adds metadata to telemetry records.

### Example

```text
service.name=emailservice
service.namespace=opentelemetry-demo
service.version=1.12.0
```

This helps identify the Email Service in observability tools.

---

## Email Processing Workflow

When a customer completes an order:

```text
Customer Places Order
          ↓
Checkout Service
          ↓
Email Service
          ↓
Generate Confirmation Email
          ↓
Send Email
          ↓
Success Response Returned
```

---

## Resource Limits

```yaml
resources:
  limits:
    memory: 100Mi
```

### Purpose

Restricts memory consumption.

### Maximum Allowed

```text
100 MiB
```

### Benefit

Protects cluster resources from excessive memory usage.

---

## Volume Mounts

```yaml
volumeMounts:
```

### Purpose

Used for attaching storage inside containers.

### Current State

No volume mounts are configured.

---

## Volumes

```yaml
volumes:
```

### Purpose

Defines storage resources available to the Pod.

### Current State

No volumes are configured.

These sections are reserved for future requirements such as:

* Persistent Volumes
* Configuration Files
* Secrets
* Shared Storage

---

## Complete Deployment Flow

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
Email Service Started
            ↓
Port 8080 Opened
            ↓
Telemetry Configuration Loaded
            ↓
Connected To OpenTelemetry Collector
            ↓
Email Service Ready
```

---

## Runtime Request Flow

```text
Checkout Service
      ↓
Email Service
      ↓
Generate Email
      ↓
Send Notification
      ↓
Return Success Response
```

---

## Summary

This Deployment file:

* Deploys the Email Service application
* Maintains one running Pod
* Exposes application port 8080
* Runs in the production environment
* Sends distributed traces to OpenTelemetry Collector
* Automatically recreates failed Pods
* Supports rolling updates and rollbacks
* Restricts memory usage to 100Mi
* Uses Kubernetes labels for service identification
* Generates email notifications for completed orders

The overall goal of this Deployment is to provide a reliable email notification service that sends order confirmations and integrates with the OpenTelemetry observability platform for monitoring and tracing.
