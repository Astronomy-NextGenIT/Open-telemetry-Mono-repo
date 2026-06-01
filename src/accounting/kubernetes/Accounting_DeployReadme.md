# Accounting Service Deployment Documentation

## Overview

This `deploy.yaml` file is responsible for deploying the **Accounting Service** in the Kubernetes cluster.

The Deployment ensures that the Accounting Service container is running, monitored, and automatically recreated if it fails. It also defines how the application should start, what image should be used, which environment variables should be provided, and which dependencies must be available before the service starts.

---

# High-Level Flow

```text
deployment.yaml
       ↓
Deployment Created
       ↓
ReplicaSet Created
       ↓
Pod Created
       ↓
Kafka Availability Checked
       ↓
Accounting Service Container Started
       ↓
Telemetry Data Sent to OpenTelemetry Collector
       ↓
Service Ready to Process Messages
```

---

# Deployment Metadata

```yaml
apiVersion: apps/v1
kind: Deployment
```

### Purpose

* `apiVersion: apps/v1` tells Kubernetes which API version to use.
* `kind: Deployment` tells Kubernetes that this resource is a Deployment.

### What Happens

When this file is applied:

```bash
kubectl apply -f deployment.yaml
```

Kubernetes creates a Deployment object that manages the Accounting Service lifecycle.

---

# Deployment Name

```yaml
metadata:
  name: opentelemetry-demo-accountingservice
```

### Purpose

Provides a unique name for the Deployment.

### What Happens

Kubernetes identifies and manages this application using this name.

---

# Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-accountingservice
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: accountingservice
  app.kubernetes.io/name: opentelemetry-demo-accountingservice
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

### Purpose

Labels are metadata attached to Kubernetes resources.

### Why They Are Important

Labels help Kubernetes:

* Identify resources
* Group related resources
* Connect Services to Pods
* Filter resources during monitoring and troubleshooting

### Example

The label:

```yaml
app.kubernetes.io/component: accountingservice
```

indicates that this Pod belongs to the Accounting Service component.

---

# Replica Configuration

```yaml
replicas: 1
```

### Purpose

Defines how many Pod instances should run.

### What Happens

Kubernetes ensures that:

```text
1 Accounting Service Pod
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

### Purpose

Stores previous Deployment versions.

### Benefit

Allows rollback if a deployment update causes issues.

Example:

```text
Version 1.12.0
       ↓
Version 1.13.0 Fails
       ↓
Rollback Possible
```

---

# Selector

```yaml
selector:
  matchLabels:
    opentelemetry.io/name: opentelemetry-demo-accountingservice
```

### Purpose

Tells the Deployment which Pods belong to it.

### What Happens

Kubernetes continuously checks:

```text
Does Pod contain this label?
       ↓
Yes
       ↓
Manage this Pod
```

---

# Pod Template

```yaml
template:
```

### Purpose

Acts as a blueprint for creating Pods.

### What Happens

Whenever Kubernetes needs a new Pod, it uses this template.

```text
Deployment
      ↓
Pod Template
      ↓
New Pod Created
```

---

# Service Account

```yaml
serviceAccountName: opentelemetry-demo
```

### Purpose

Defines the Kubernetes Service Account used by the Pod.

### Why Needed

Provides permissions for interacting with Kubernetes resources securely.

---

# Main Application Container

```yaml
containers:
  - name: accountingservice
```

### Purpose

Defines the primary application container.

### What Happens

This container runs the Accounting Service application.

---

# Container Image

```yaml
image: ghcr.io/open-telemetry/demo:1.12.0-accountingservice
```

### Purpose

Specifies the Docker image used to start the container.

### What Happens

Kubernetes pulls the image from:

```text
GitHub Container Registry (GHCR)
```

and starts the application.

---

# Image Pull Policy

```yaml
imagePullPolicy: IfNotPresent
```

### Purpose

Controls image download behavior.

### What Happens

```text
Image Exists on Node
       ↓
Use Existing Image

Image Not Found
       ↓
Download From Registry
```

This reduces unnecessary downloads.

---

# Environment Variables

Environment variables provide runtime configuration to the application.

---

## OTEL_SERVICE_NAME

```yaml
OTEL_SERVICE_NAME
```

### Purpose

Automatically sets the service name using Pod labels.

### Result

```text
accountingservice
```

This name appears in telemetry dashboards.

---

## OTEL_COLLECTOR_NAME

```yaml
OTEL_COLLECTOR_NAME=opentelemetry-demo-otelcol
```

### Purpose

Specifies the OpenTelemetry Collector service name.

### What Happens

Accounting Service sends telemetry data to this collector.

---

## Metrics Temporality

```yaml
OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE=cumulative
```

### Purpose

Controls how metrics are reported.

### Meaning

Metrics continuously accumulate values over time.

---

## Kafka Address

```yaml
KAFKA_SERVICE_ADDR=opentelemetry-demo-kafka:9092
```

### Purpose

Defines Kafka broker location.

### What Happens

Accounting Service communicates with Kafka using:

```text
Host: opentelemetry-demo-kafka
Port: 9092
```

---

## OpenTelemetry Endpoint

```yaml
OTEL_EXPORTER_OTLP_ENDPOINT=http://$(OTEL_COLLECTOR_NAME):4318
```

### Purpose

Defines where telemetry data should be exported.

### Final Value

```text
http://opentelemetry-demo-otelcol:4318
```

### What Happens

The service sends:

* Metrics
* Traces
* Logs

to the OpenTelemetry Collector.

---

## Resource Attributes

```yaml
OTEL_RESOURCE_ATTRIBUTES
```

### Purpose

Adds metadata to telemetry data.

### Information Included

```text
Service Name
Namespace
Version
```

Example:

```text
service.name=accountingservice
service.namespace=opentelemetry-demo
service.version=1.12.0
```

This information appears in observability tools.

---

# Resource Limits

```yaml
resources:
  limits:
    memory: 120Mi
```

### Purpose

Restricts maximum memory usage.

### What Happens

```text
Maximum Memory Allowed
       ↓
120 MiB
```

This prevents a single container from consuming excessive memory.

---

# Init Container

```yaml
initContainers:
```

### Purpose

Runs before the main application container starts.

---

## Wait for Kafka

```yaml
until nc -z -v -w30 opentelemetry-demo-kafka 9092
```

### What Happens

The init container repeatedly checks:

```text
Is Kafka Available?
```

If No:

```text
Wait 2 Seconds
       ↓
Check Again
```

If Yes:

```text
Start Accounting Service
```

---

# Why Init Container Is Important

Without this check:

```text
Accounting Service Starts
       ↓
Kafka Not Ready
       ↓
Connection Failure
       ↓
Application Errors
```

With init container:

```text
Wait For Kafka
       ↓
Kafka Ready
       ↓
Start Accounting Service
       ↓
Successful Connection
```

This improves deployment reliability.

---

# Complete Deployment Flow

```text
kubectl apply deployment.yaml
            ↓
Deployment Created
            ↓
ReplicaSet Created
            ↓
Pod Created
            ↓
Init Container Starts
            ↓
Checks Kafka Availability
            ↓
Kafka Ready
            ↓
Main Accounting Service Container Starts
            ↓
Environment Variables Loaded
            ↓
Telemetry Configuration Applied
            ↓
Application Connects To Kafka
            ↓
Application Sends Traces/Metrics To OTEL Collector
            ↓
Accounting Service Ready
```

---

# Summary

This Deployment file:

* Deploys the Accounting Service application
* Maintains one running Pod
* Uses a GitHub Container Registry image
* Connects to Kafka
* Sends telemetry data to OpenTelemetry Collector
* Automatically recreates failed Pods
* Waits for Kafka before starting
* Applies memory limits for resource control
* Supports rollback through Deployment history

The overall goal of this Deployment is to ensure that the Accounting Service starts reliably, connects successfully to Kafka, and continuously sends observability data to the OpenTelemetry platform.
