# Fraud Detection Service Deployment (`deploy.yaml`) Documentation

## Overview

This Kubernetes Deployment file is responsible for deploying the **Fraud Detection Service** of the OpenTelemetry Demo application.

The Fraud Detection Service analyzes transactions and helps identify potentially fraudulent activities. It communicates with other services through Kafka and sends 
telemetry data (metrics, logs, and traces) to the OpenTelemetry Collector for monitoring and observability.

---

# 1. API Version and Resource Type

```yaml
apiVersion: apps/v1
kind: Deployment
```

### What it means

* `apps/v1` is the Kubernetes API version used for Deployments.
* `Deployment` manages application Pods automatically.

### Why it is used

Deployment provides:

* Pod creation
* Pod updates
* Rollbacks
* Self-healing
* Scaling

If a Pod crashes, Kubernetes automatically creates a new one.

---

# 2. Metadata

```yaml
metadata:
  name: opentelemetry-demo-frauddetectionservice
```

### What it means

The deployment name is:

```text
opentelemetry-demo-frauddetectionservice
```

This name uniquely identifies the deployment inside the Kubernetes cluster.

---

# 3. Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-frauddetectionservice
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: frauddetectionservice
  app.kubernetes.io/name: opentelemetry-demo-frauddetectionservice
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

## Purpose of Labels

Labels are key-value pairs attached to Kubernetes resources.

### Why labels are important

They help Kubernetes:

* Identify resources
* Group related resources
* Select Pods
* Monitor applications

### Example

```yaml
app.kubernetes.io/component: frauddetectionservice
```

This label tells Kubernetes that this Pod belongs to the Fraud Detection Service.

---

# 4. Deployment Specification

```yaml
spec:
```

Defines how Kubernetes should run the application.

---

## 4.1 Number of Replicas

```yaml
replicas: 1
```

### What it means

Only **one Pod** of the Fraud Detection Service will run.

### Example

```text
Fraud Detection Service
        |
        └── Pod-1
```

If Pod-1 fails, Kubernetes creates a replacement automatically.

---

## 4.2 Revision History

```yaml
revisionHistoryLimit: 10
```

### What it means

Kubernetes keeps the last **10 deployment versions**.

### Why it is useful

If a deployment fails after an update, you can roll back to a previous version.

---

# 5. Pod Selector

```yaml
selector:
  matchLabels:
    opentelemetry.io/name: opentelemetry-demo-frauddetectionservice
```

### What it means

The Deployment manages Pods having this label.

### How it works

Kubernetes looks for Pods with:

```yaml
opentelemetry.io/name: opentelemetry-demo-frauddetectionservice
```

and treats them as part of this Deployment.

---

# 6. Pod Template

```yaml
template:
```

This section describes how Pods should be created.

---

## 6.1 Pod Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-frauddetectionservice
  app.kubernetes.io/component: frauddetectionservice
```

### Purpose

These labels are applied to every Pod created by this Deployment.

---

# 7. Service Account

```yaml
serviceAccountName: opentelemetry-demo
```

### What it means

The Pod uses the Kubernetes Service Account:

```text
opentelemetry-demo
```

### Why it is needed

It provides permissions to access Kubernetes resources securely.

---

# 8. Main Container

```yaml
containers:
  - name: frauddetectionservice
```

### What it means

This is the main application container running the Fraud Detection Service.

---

## 8.1 Container Image

```yaml
image: ghcr.io/open-telemetry/demo:1.12.0-frauddetectionservice
```

### What it means

The container image is pulled from GitHub Container Registry.

### Image Breakdown

```text
ghcr.io
   |
   └── open-telemetry
         |
         └── demo
               |
               └── 1.12.0-frauddetectionservice
```

### Version

```text
1.12.0
```

is the application version being deployed.

---

## 8.2 Image Pull Policy

```yaml
imagePullPolicy: IfNotPresent
```

### What it means

Kubernetes will:

* Use local image if available
* Download image only if it does not exist

### Benefit

Reduces deployment time and network usage.

---

# 9. Environment Variables

Environment variables provide runtime configuration to the application.

---

## 9.1 Service Name

```yaml
- name: OTEL_SERVICE_NAME
```

Value comes from:

```yaml
metadata.labels['app.kubernetes.io/component']
```

Result:

```text
frauddetectionservice
```

### Purpose

Used by OpenTelemetry to identify this service in traces and metrics.

---

## 9.2 OpenTelemetry Collector Name

```yaml
- name: OTEL_COLLECTOR_NAME
  value: opentelemetry-demo-otelcol
```

### Purpose

Specifies the Collector service responsible for receiving telemetry data.

---

## 9.3 Metrics Temporality

```yaml
- name: OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE
  value: cumulative
```

### Purpose

Metrics are reported as cumulative values.

### Example

```text
Requests Processed

1st minute = 100
2nd minute = 250
3rd minute = 500
```

The total count keeps increasing.

---

## 9.4 Kafka Server Address

```yaml
- name: KAFKA_SERVICE_ADDR
  value: opentelemetry-demo-kafka:9092
```

### Purpose

Tells the application where Kafka is running.

### Breakdown

```text
Service Name : opentelemetry-demo-kafka
Port         : 9092
```

### Why Kafka is needed

The Fraud Detection Service consumes transaction events from Kafka.

---

## 9.5 Feature Flag Service Host

```yaml
- name: FLAGD_HOST
  value: opentelemetry-demo-flagd
```

### Purpose

Provides the hostname of the Flagd feature flag service.

---

## 9.6 Feature Flag Port

```yaml
- name: FLAGD_PORT
  value: "8013"
```

### Purpose

Specifies the communication port for Flagd.

---

## 9.7 OpenTelemetry Endpoint

```yaml
- name: OTEL_EXPORTER_OTLP_ENDPOINT
  value: http://$(OTEL_COLLECTOR_NAME):4318
```

### Resolved Value

```text
http://opentelemetry-demo-otelcol:4318
```

### Purpose

Sends traces, logs, and metrics to the OpenTelemetry Collector.

---

## 9.8 Resource Attributes

```yaml
- name: OTEL_RESOURCE_ATTRIBUTES
```

Value:

```text
service.name=frauddetectionservice,
service.namespace=opentelemetry-demo,
service.version=1.12.0
```

### Purpose

Adds metadata to telemetry data.

### Benefits

Helps monitoring tools identify:

* Service Name
* Namespace
* Version

---

# 10. Resource Limits

```yaml
resources:
  limits:
    memory: 300Mi
```

### What it means

Maximum memory allowed:

```text
300 MiB
```

### Why it is important

Prevents the container from consuming excessive memory and affecting other workloads.

---

# 11. Volume Mounts

```yaml
volumeMounts:
```

Currently no volume mounts are configured.

### Meaning

The application does not require persistent storage.

---

# 12. Init Container

Before the main application starts, Kubernetes runs an Init Container.

```yaml
initContainers:
```

---

## Wait for Kafka

```yaml
name: wait-for-kafka
```

Image:

```yaml
busybox:latest
```

Command:

```sh
until nc -z -v -w30 opentelemetry-demo-kafka 9092;
do
  echo waiting for kafka;
  sleep 2;
done;
```

### What it does

Checks whether Kafka is available.

### Process

```text
Start Init Container
        |
        v
Check Kafka
        |
   Available?
    /      \
   No       Yes
   |         |
Wait 2 sec   Start Application
```

### Why it is needed

The Fraud Detection Service depends on Kafka.

Without this check:

```text
Application starts
      |
Kafka not ready
      |
Connection Failure
      |
Application Error
```

With the Init Container:

```text
Wait for Kafka
      |
Kafka Ready
      |
Application Starts Successfully
```

---

# 13. Volumes

```yaml
volumes:
```

Currently no volumes are defined.

### Meaning

The service runs completely from the container image and does not need external storage.

---

# Deployment Flow

```text
Deployment Created
        |
        v
Create Pod
        |
        v
Run Init Container
(wait-for-kafka)
        |
        v
Kafka Available?
        |
       Yes
        |
        v
Start Fraud Detection Service
        |
        v
Connect to Kafka
        |
        v
Send Telemetry to OpenTelemetry Collector
        |
        v
Process Fraud Detection Events
```

# Summary

This Deployment:

* Creates 1 Fraud Detection Service Pod.
* Uses image `ghcr.io/open-telemetry/demo:1.12.0-frauddetectionservice`.
* Waits for Kafka before starting.
* Connects to Kafka on port `9092`.
* Uses Flagd for feature flags.
* Sends logs, metrics, and traces to the OpenTelemetry Collector.
* Limits memory usage to `300Mi`.
* Uses Kubernetes Deployment features such as self-healing, updates, and rollbacks.

In simple terms, this file tells Kubernetes: **"Run one Fraud Detection Service container, wait until Kafka is available, connect it to the OpenTelemetry monitoring stack, and keep it running automatically."**
