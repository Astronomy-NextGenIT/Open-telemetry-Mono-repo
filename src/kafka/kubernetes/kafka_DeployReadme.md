# Kafka Deployment (`deploy.yaml`) Documentation

## Overview

This Kubernetes Deployment file deploys the **Kafka Service** used by the OpenTelemetry Demo application.

Kafka acts as a **message broker** that allows different microservices to exchange data asynchronously. Instead of services communicating directly, they can send messages to Kafka topics and consume them when needed.

### Example

In the OpenTelemetry Demo:

```text
Checkout Service
        |
        v
      Kafka
        |
        v
Fraud Detection Service
```

The Checkout Service publishes order events to Kafka, and the Fraud Detection Service consumes those events for fraud analysis.

---

# Kafka Architecture

```text
Frontend
    |
    v
Checkout Service
    |
    v
Kafka
    |
    +------------------+
    |                  |
    v                  v
Fraud Detection   Other Consumers
```

Kafka acts as a central event streaming platform.

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
* Rollbacks
* Scaling support

---

# 2. Metadata

```yaml
metadata:
  name: opentelemetry-demo-kafka
```

## Purpose

Deployment name:

```text
opentelemetry-demo-kafka
```

Used by Kubernetes to manage the Kafka Deployment.

---

# 3. Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-kafka
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: kafka
  app.kubernetes.io/name: opentelemetry-demo-kafka
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

## Purpose

Labels help Kubernetes:

* Identify resources
* Select Pods
* Group components
* Enable monitoring

Example:

```yaml
app.kubernetes.io/component: kafka
```

Indicates that this component is Kafka.

---

# 4. Deployment Specification

```yaml
spec:
```

Defines how Kafka should run.

---

## 4.1 Replicas

```yaml
replicas: 1
```

### Purpose

Runs one Kafka Pod.

```text
Kafka Deployment
        |
        v
     Kafka Pod
```

If the Pod crashes, Kubernetes automatically creates a replacement.

---

## 4.2 Revision History

```yaml
revisionHistoryLimit: 10
```

### Purpose

Stores the last 10 deployment versions.

Allows rollback during failed upgrades.

---

# 5. Selector

```yaml
selector:
  matchLabels:
    opentelemetry.io/name: opentelemetry-demo-kafka
```

## Purpose

The Deployment manages Pods matching this label.

---

# 6. Pod Template

```yaml
template:
```

Defines how Kafka Pods should be created.

---

## Pod Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-kafka
```

These labels are applied to all Kafka Pods.

---

# 7. Service Account

```yaml
serviceAccountName: opentelemetry-demo
```

## Purpose

Kafka runs using the Service Account:

```text
opentelemetry-demo
```

This provides secure Kubernetes access if needed.

---

# 8. Kafka Container

```yaml
containers:
  - name: kafka
```

This is the main Kafka container.

---

## 8.1 Container Image

```yaml
image: ghcr.io/open-telemetry/demo:1.12.0-kafka
```

### Breakdown

```text
Registry : ghcr.io
Project  : open-telemetry/demo
Version  : 1.12.0
Service  : kafka
```

### Purpose

Contains the Kafka broker software used by the demo application.

---

## 8.2 Image Pull Policy

```yaml
imagePullPolicy: IfNotPresent
```

### Purpose

* Uses local image if available
* Pulls image only when needed

### Benefit

Faster startup and reduced bandwidth usage.

---

# 9. Container Ports

Kafka exposes two ports.

---

## 9.1 Client Port

```yaml
containerPort: 9092
name: plaintext
```

### Purpose

Used by applications to connect to Kafka.

Examples:

* Checkout Service
* Fraud Detection Service
* Other producers and consumers

Flow:

```text
Application
      |
      v
Kafka :9092
```

---

## 9.2 Controller Port

```yaml
containerPort: 9093
name: controller
```

### Purpose

Used internally by Kafka for cluster controller operations.

In Kafka KRaft mode, this port manages broker coordination.

---

# 10. OpenTelemetry Configuration

## Service Name

```yaml
OTEL_SERVICE_NAME
```

Resolved value:

```text
kafka
```

### Purpose

Identifies Kafka telemetry data.

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

Exports metrics as cumulative values.

---

# 11. Kafka Advertised Listeners

```yaml
KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://opentelemetry-demo-kafka:9092
```

## Purpose

Tells clients how to connect to Kafka.

### Advertised Address

```text
opentelemetry-demo-kafka:9092
```

When a client connects, Kafka returns this address.

---

### Example

```text
Fraud Detection Service
         |
         v
opentelemetry-demo-kafka:9092
```

---

# 12. OpenTelemetry Export Endpoint

```yaml
OTEL_EXPORTER_OTLP_ENDPOINT=http://$(OTEL_COLLECTOR_NAME):4318
```

Resolved value:

```text
http://opentelemetry-demo-otelcol:4318
```

### Purpose

Kafka sends telemetry data to the OpenTelemetry Collector.

Telemetry includes:

* Metrics
* Logs
* Traces

---

# 13. Kafka JVM Heap Configuration

```yaml
KAFKA_HEAP_OPTS=-Xmx400M -Xms400M
```

## Purpose

Configures Java memory allocation.

### Breakdown

#### Initial Heap

```text
-Xms400M
```

Allocates 400 MB at startup.

#### Maximum Heap

```text
-Xmx400M
```

Limits heap to 400 MB.

---

### Why Fixed Heap?

```text
Minimum = 400 MB
Maximum = 400 MB
```

Benefits:

* Predictable memory usage
* Stable performance
* Reduced garbage collection overhead

---

# 14. Resource Attributes

```yaml
OTEL_RESOURCE_ATTRIBUTES
```

Resolved value:

```text
service.name=kafka,
service.namespace=opentelemetry-demo,
service.version=1.12.0
```

### Purpose

Adds metadata to telemetry.

Used by observability tools to identify:

* Service Name
* Namespace
* Version

---

# 15. Resource Limits

```yaml
resources:
  limits:
    memory: 600Mi
```

## Purpose

Maximum memory allowed:

```text
600 MiB
```

### Why Important

Prevents Kafka from consuming excessive cluster memory.

---

# 16. Security Context

```yaml
securityContext:
  runAsGroup: 1000
  runAsNonRoot: true
  runAsUser: 1000
```

## Purpose

Improves container security.

### runAsNonRoot

```yaml
runAsNonRoot: true
```

Prevents Kafka from running as root.

---

### runAsUser

```yaml
runAsUser: 1000
```

Runs Kafka using User ID 1000.

---

### runAsGroup

```yaml
runAsGroup: 1000
```

Runs Kafka using Group ID 1000.

---

### Benefit

Reduces security risks if the container is compromised.

---

# 17. Volume Mounts

```yaml
volumeMounts:
```

No volume mounts are configured.

### Meaning

Kafka data is not stored on persistent volumes in this demo environment.

---

# 18. Volumes

```yaml
volumes:
```

No volumes are defined.

### Meaning

This Kafka deployment is intended for demonstration purposes, not production workloads.

---

# Kafka Message Flow

```text
Checkout Service
        |
        | Publish Event
        v
     Kafka Topic
        |
        | Consume Event
        v
Fraud Detection Service
```

---

# Complete Deployment Flow

```text
Deployment Created
        |
        v
Create Kafka Pod
        |
        v
Start Kafka Broker
        |
        v
Listen on Port 9092
        |
        v
Accept Producer Messages
        |
        v
Store Events
        |
        v
Deliver Events To Consumers
        |
        v
Send Telemetry To OTEL Collector
```

---

# Role of Kafka in OpenTelemetry Demo

```text
Frontend
    |
    v
Checkout Service
    |
    v
Kafka
    |
    v
Fraud Detection Service
```

Kafka enables asynchronous communication between services.

Instead of waiting for an immediate response, services exchange events through Kafka topics.

---

# Summary

This Deployment:

* Creates one Kafka Pod.
* Uses image `ghcr.io/open-telemetry/demo:1.12.0-kafka`.
* Exposes Kafka client traffic on port `9092`.
* Uses port `9093` for Kafka controller operations.
* Advertises itself as `opentelemetry-demo-kafka:9092`.
* Sends telemetry data to the OpenTelemetry Collector.
* Allocates a fixed JVM heap of `400 MB`.
* Limits memory usage to `600Mi`.
* Runs as a non-root user (`UID 1000`).
* Supports Kubernetes self-healing and rollbacks.

## In Simple Terms

Kafka is the **event messaging backbone** of the OpenTelemetry Demo application. It allows services such as Checkout and Fraud Detection to exchange messages asynchronously, making the application more scalable and resilient. This Deployment ensures Kafka is always running, properly monitored, and securely configured inside the Kubernetes cluster.
