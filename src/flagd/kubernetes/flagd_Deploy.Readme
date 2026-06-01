# Flagd Deployment Documentation

## Overview

This `deploy.yaml` file deploys the **Flagd Service** and **Flagd UI** in Kubernetes.

Unlike most deployments in the OpenTelemetry Demo, this Deployment contains **two containers running inside the same Pod**:

1. **Flagd Container** – Feature Flag Engine
2. **Flagd UI Container** – Web Interface for managing and viewing feature flags

The purpose of this deployment is to provide **Feature Flag Management** for the entire application.

Feature flags allow developers to enable or disable features without redeploying applications.

---

# What is Flagd?

Flagd is an OpenFeature-compatible feature flag service.

Instead of changing application code:

```text
Deploy New Code
        ↓
Restart Application
```

Developers can simply change a flag:

```text
Update Feature Flag
        ↓
Feature Enabled/Disabled
        ↓
No Application Restart Needed
```

---

# High-Level Architecture

```text
Frontend
     ↓
Feature Request
     ↓
Flagd Service
     ↓
Read Feature Flags
     ↓
Return Feature Status
     ↓
Application Behavior Changes
```

---

# Deployment Resource

```yaml
apiVersion: apps/v1
kind: Deployment
```

## Purpose

Creates and manages the Flagd Pod.

---

# Deployment Name

```yaml
metadata:
  name: opentelemetry-demo-flagd
```

## Purpose

Unique name of the deployment.

---

# Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-flagd
  app.kubernetes.io/component: flagd
```

## Purpose

Used for:

* Service discovery
* Monitoring
* Pod selection
* Resource grouping

---

# Replica Configuration

```yaml
replicas: 1
```

## Purpose

Maintains one Flagd Pod.

### Failure Recovery

```text
Pod Crashes
      ↓
Deployment Detects Failure
      ↓
New Pod Created
```

---

# Revision History

```yaml
revisionHistoryLimit: 10
```

## Purpose

Stores 10 previous deployment versions.

Supports rollback operations.

---

# Selector

```yaml
selector:
  matchLabels:
    opentelemetry.io/name: opentelemetry-demo-flagd
```

## Purpose

Defines which Pods belong to this deployment.

---

# Pod Template

```yaml
template:
```

## Purpose

Acts as the blueprint for Pod creation.

---

# Service Account

```yaml
serviceAccountName: opentelemetry-demo
```

## Purpose

Provides Kubernetes permissions to the Pod.

---

# Container 1 – Flagd

```yaml
containers:
  - name: flagd
```

## Purpose

Runs the actual Feature Flag Engine.

---

# Container Image

```yaml
image: ghcr.io/open-feature/flagd:v0.11.1
```

## Purpose

Downloads and runs the Flagd application.

---

# Startup Command

```yaml
command:
  - /flagd-build
  - start
  - --uri
  - file:./etc/flagd/demo.flagd.json
```

## Purpose

Starts Flagd using a configuration file.

### Runtime Flow

```text
Flagd Starts
      ↓
Reads demo.flagd.json
      ↓
Loads Feature Flags
      ↓
Ready To Serve Requests
```

---

# Flagd Port

```yaml
containerPort: 8013
```

## Purpose

Port used by applications to communicate with Flagd.

### Flow

```text
Application
      ↓
Flagd:8013
      ↓
Feature Flag Response
```

---

# Flagd Environment Variables

## OTEL_SERVICE_NAME

Automatically sets service name:

```text
flagd
```

Used in telemetry data.

---

## OTEL_COLLECTOR_NAME

```yaml
OTEL_COLLECTOR_NAME=opentelemetry-demo-otelcol
```

Defines OpenTelemetry Collector location.

---

## Metrics Temporality

```yaml
OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE=cumulative
```

Controls metric aggregation.

---

## FLAGD_METRICS_EXPORTER

```yaml
FLAGD_METRICS_EXPORTER=otel
```

## Purpose

Exports Flagd metrics through OpenTelemetry.

---

## FLAGD_OTEL_COLLECTOR_URI

```yaml
FLAGD_OTEL_COLLECTOR_URI=$(OTEL_COLLECTOR_NAME):4317
```

## Purpose

Specifies where metrics should be sent.

### Flow

```text
Flagd
   ↓
Generate Metrics
   ↓
OTEL Collector
   ↓
Monitoring Platform
```

---

# Flagd Resource Limit

```yaml
memory: 75Mi
```

Maximum memory usage:

```text
75 MiB
```

---

# Flagd Volume Mount

```yaml
mountPath: /etc/flagd
```

## Purpose

Makes feature flag configuration files available.

---

# Container 2 – Flagd UI

```yaml
containers:
  - name: flagdui
```

## Purpose

Provides a web interface for viewing and managing feature flags.

---

# Flagd UI Image

```yaml
image: ghcr.io/open-telemetry/demo:1.12.0-flagdui
```

## Purpose

Runs the Flagd UI application.

---

# Flagd UI Port

```yaml
containerPort: 4000
```

## Purpose

Hosts the Flagd management interface.

### Flow

```text
Browser
    ↓
Flagd UI
    ↓
Port 4000
```

---

# Flagd UI OpenTelemetry Endpoint

```yaml
OTEL_EXPORTER_OTLP_ENDPOINT=http://opentelemetry-demo-otelcol:4318
```

## Purpose

Exports UI telemetry data.

---

# Flagd UI Resource Limit

```yaml
memory: 75Mi
```

Maximum memory usage:

```text
75 MiB
```

---

# Flagd UI Volume Mount

```yaml
mountPath: /app/data
```

## Purpose

Provides access to feature flag configuration files.

---

# Init Container

```yaml
initContainers:
```

Unlike normal containers, Init Containers run before the application starts.

---

# Init Container Name

```yaml
name: init-config
```

## Purpose

Prepares configuration files before Flagd starts.

---

# Init Container Image

```yaml
image: busybox
```

Lightweight Linux container used for setup tasks.

---

# Init Container Command

```yaml
cp /config-ro/demo.flagd.json /config-rw/demo.flagd.json
```

## Purpose

Copies configuration file from read-only storage to writable storage.

### Flow

```text
ConfigMap
      ↓
Read Only Volume
      ↓
Init Container
      ↓
Copy File
      ↓
Writable Volume
      ↓
Flagd Starts
```

---

# Volumes

This deployment uses two volumes.

---

# Volume 1 – Writable Volume

```yaml
emptyDir: {}
```

## Purpose

Creates temporary writable storage.

```yaml
name: config-rw
```

### Used By

* Flagd
* Flagd UI

---

# Volume 2 – ConfigMap Volume

```yaml
configMap:
  name: opentelemetry-demo-flagd-config
```

## Purpose

Provides feature flag configuration files.

```text
ConfigMap
      ↓
demo.flagd.json
      ↓
Feature Definitions
```

### Mounted As

```yaml
name: config-ro
```

Read-only volume.

---

# Complete Startup Flow

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
Reads ConfigMap
            ↓
Copies demo.flagd.json
            ↓
Init Container Completes
            ↓
Flagd Container Starts
            ↓
Loads Feature Flags
            ↓
Port 8013 Opened
            ↓
Flagd UI Starts
            ↓
Port 4000 Opened
            ↓
Telemetry Connected
            ↓
Flag Management System Ready
```

---

# Runtime Feature Flag Flow

```text
Application
      ↓
Feature Check Request
      ↓
Flagd Service
      ↓
Read demo.flagd.json
      ↓
Feature Enabled?
      ↓
Yes / No Response
      ↓
Application Behavior Updated
```

---

# Why Flagd is Important

Without Flagd:

```text
Change Feature
      ↓
Modify Code
      ↓
Build New Image
      ↓
Deploy Application
```

With Flagd:

```text
Change Feature Flag
      ↓
Save Configuration
      ↓
Feature Updated Immediately
```

---

# Summary

This Deployment file:

* Deploys the Flagd Feature Flag Engine
* Deploys the Flagd UI management interface
* Runs two containers inside a single Pod
* Loads feature flags from a ConfigMap
* Uses an Init Container to prepare configuration files
* Exposes Flagd on port 8013
* Exposes Flagd UI on port 4000
* Integrates with OpenTelemetry
* Exports metrics and telemetry data
* Maintains one running Pod
* Automatically recreates failed Pods
* Supports rolling updates and rollbacks

The overall goal of this Deployment is to provide centralized feature flag management, allowing applications to enable or disable features dynamically without requiring new deployments.
