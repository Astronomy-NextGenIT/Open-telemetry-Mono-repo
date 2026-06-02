# Cart Service Deployment Documentation

## Overview

This `deploy.yaml` file is responsible for deploying the **Cart Service** in the Kubernetes cluster.

The Cart Service manages shopping cart operations for users. It stores and retrieves cart information using **Valkey** (Redis-compatible in-memory datastore), 
integrates with **Flagd** for feature flags, and sends telemetry data to the **OpenTelemetry Collector**.

This Deployment ensures the Cart Service is always running, properly connected to its dependencies, and automatically restarted if failures occur.

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
Wait For Valkey
      ↓
Cart Service Container Started
      ↓
Port 8080 Opened
      ↓
Connect To Valkey
      ↓
Connect To Flagd
      ↓
Send Telemetry To OpenTelemetry Collector
      ↓
Cart Service Ready
```

---

# Deployment Resource

```yaml
apiVersion: apps/v1
kind: Deployment
```

## Purpose

Defines a Kubernetes Deployment resource that manages Cart Service Pods.

### What Happens

```text
kubectl apply deploy.yaml
           ↓
Deployment Created
           ↓
Pod Lifecycle Managed Automatically
```

---

# Deployment Name

```yaml
metadata:
  name: opentelemetry-demo-cartservice
```

## Purpose

Provides a unique identifier for the Cart Service Deployment.

---

# Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-cartservice
  app.kubernetes.io/component: cartservice
  app.kubernetes.io/name: opentelemetry-demo-cartservice
```

## Purpose

Labels help Kubernetes identify and group resources.

### Benefits

* Pod selection
* Service discovery
* Monitoring
* Troubleshooting
* Resource organization

---

# Replica Configuration

```yaml
replicas: 1
```

## Purpose

Ensures one Cart Service Pod is always running.

### Failure Recovery

```text
Pod Crashes
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

Stores up to 10 previous Deployment versions.

### Benefit

Supports rollback during failed updates.

---

# Selector

```yaml
selector:
  matchLabels:
    opentelemetry.io/name: opentelemetry-demo-cartservice
```

## Purpose

Defines which Pods belong to this Deployment.

### Flow

```text
Deployment
      ↓
Find Pods With Matching Labels
      ↓
Manage Those Pods
```

---

# Pod Template

```yaml
template:
```

## Purpose

Acts as the blueprint used to create Cart Service Pods.

---

# Service Account

```yaml
serviceAccountName: opentelemetry-demo
```

## Purpose

Provides the Pod with Kubernetes permissions through the Service Account.

---

# Main Application Container

```yaml
containers:
  - name: cartservice
```

## Purpose

Defines the Cart Service application container.

---

# Container Image

```yaml
image: ghcr.io/open-telemetry/demo:1.12.0-cartservice
```

## Purpose

Specifies the Docker image used to run the Cart Service.

### What Happens

```text
GitHub Container Registry
          ↓
Image Pulled
          ↓
Container Started
```

---

# Image Pull Policy

```yaml
imagePullPolicy: IfNotPresent
```

## Purpose

Prevents unnecessary image downloads.

### Behavior

```text
Image Exists
      ↓
Use Existing Copy

Image Missing
      ↓
Download From Registry
```

---

# Container Port

```yaml
ports:
  - containerPort: 8080
```

## Purpose

Exposes application port 8080 inside the container.

### What Happens

The Cart Service listens for requests on:

```text
Port 8080
```

---

# Environment Variables

Environment variables provide runtime configuration.

---

## OTEL_SERVICE_NAME

```yaml
OTEL_SERVICE_NAME
```

### Purpose

Automatically sets the service name using Kubernetes labels.

### Result

```text
cartservice
```

Used in traces, metrics, and logs.

---

## CART_SERVICE_PORT

```yaml
CART_SERVICE_PORT=8080
```

### Purpose

Defines the application's listening port.

---

## ASPNETCORE_URLS

```yaml
ASPNETCORE_URLS=http://*:8080
```

## Purpose

Configures the ASP.NET Core application to listen on all network interfaces.

### Result

```text
Any Incoming Request
        ↓
Port 8080
        ↓
Cart Service
```

---

# Valkey Configuration

## VALKEY_ADDR

```yaml
VALKEY_ADDR=opentelemetry-demo-valkey:6379
```

### Purpose

Specifies the Valkey server address.

### What Is Valkey?

Valkey is an in-memory key-value datastore compatible with Redis.

The Cart Service uses it to:

* Store shopping carts
* Retrieve shopping carts
* Maintain user cart sessions

### Communication Flow

```text
Cart Service
      ↓
Valkey
      ↓
Port 6379
```

---

# Feature Flag Configuration

## FLAGD_HOST

```yaml
FLAGD_HOST=opentelemetry-demo-flagd
```

## FLAGD_PORT

```yaml
FLAGD_PORT=8013
```

### Purpose

Connects the Cart Service to the Flagd feature management service.

### Usage

Feature flags allow:

```text
Enable Feature
Disable Feature
Test New Features
Control Behavior Without Redeployment
```

---

# OpenTelemetry Configuration

## OTEL_COLLECTOR_NAME

```yaml
OTEL_COLLECTOR_NAME=opentelemetry-demo-otelcol
```

Defines the OpenTelemetry Collector service.

---

## OTEL_EXPORTER_OTLP_ENDPOINT

```yaml
OTEL_EXPORTER_OTLP_ENDPOINT=http://opentelemetry-demo-otelcol:4317
```

### Purpose

Specifies where telemetry data should be sent.

### Data Exported

```text
Traces
Metrics
Logs
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
service.name=cartservice
service.namespace=opentelemetry-demo
service.version=1.12.0
```

This information appears in observability dashboards.

---

# Resource Limits

```yaml
resources:
  limits:
    memory: 160Mi
```

## Purpose

Limits memory consumption.

### Maximum Allowed

```text
160 MiB
```

### Benefit

Protects cluster resources from excessive usage.

---

# Init Container

```yaml
initContainers:
```

## Purpose

Runs before the main Cart Service container starts.

---

# Wait For Valkey

```yaml
until nc -z -v -w30 opentelemetry-demo-valkey 6379
```

## Purpose

Checks whether Valkey is available before starting the Cart Service.

### Why Needed

Cart Service depends on Valkey.

Without this check:

```text
Cart Service Starts
        ↓
Valkey Not Ready
        ↓
Connection Failure
        ↓
Application Errors
```

With init container:

```text
Check Valkey
      ↓
Valkey Ready?
      ↓
Yes
      ↓
Start Cart Service
```

### Retry Logic

```text
Valkey Not Ready
      ↓
Wait 2 Seconds
      ↓
Check Again
```

This continues until Valkey becomes available.

---

# Volumes

```yaml
volumes:
volumeMounts:
```

## Purpose

Provide storage to containers.

### Current State

No volumes are currently configured.

These sections are reserved for future storage requirements.

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
Init Container Starts
            ↓
Checks Valkey Availability
            ↓
Valkey Ready
            ↓
Main Cart Service Container Starts
            ↓
Port 8080 Opened
            ↓
Connects To Valkey
            ↓
Connects To Flagd
            ↓
Telemetry Configuration Loaded
            ↓
Connects To OpenTelemetry Collector
            ↓
Metrics, Traces And Logs Exported
            ↓
Cart Service Ready
```

---

# Service Communication Flow

```text
Frontend
    ↓
Cart Service
    ↓
Port 8080
    ↓
Store/Retrieve Cart Data
    ↓
Valkey
    ↓
Response Returned
```

---

# Summary

This Deployment file:

* Deploys the Cart Service application
* Maintains one running Pod
* Exposes application port 8080
* Uses Valkey as the cart datastore
* Waits for Valkey before startup
* Connects to Flagd for feature management
* Sends telemetry to OpenTelemetry Collector
* Automatically recreates failed Pods
* Supports rollback through Deployment history
* Restricts memory usage to 160Mi

The overall goal of this Deployment is to ensure that the Cart Service starts reliably, connects successfully to Valkey, stores user cart data efficiently, and continuously exports observability data to the OpenTelemetry platform.
