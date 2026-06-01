# Kafka Service (`service.yaml`) Documentation

## Overview

This Kubernetes Service file creates a stable network endpoint for the **Kafka Broker** running in the OpenTelemetry Demo application.

Kafka is used as the event streaming and messaging platform that allows microservices to communicate asynchronously.

Since Pod IP addresses can change whenever a Pod restarts, the Service provides a permanent DNS name that applications can use to connect to Kafka.

```text
Checkout Service
        |
        v
opentelemetry-demo-kafka
        |
        v
Kafka Pod
        |
        v
Fraud Detection Service
```

Without this Service, applications would need to know the Kafka Pod IP address, which is not reliable.

---

# 1. API Version and Resource Type

```yaml
apiVersion: v1
kind: Service
```

## Purpose

### apiVersion

```yaml
v1
```

Uses Kubernetes Core API resources.

### kind

```yaml
Service
```

Creates a Kubernetes Service.

### Why Service is Needed

Pods are temporary.

Example:

```text
Kafka Pod
IP = 10.10.1.5
```

After restart:

```text
Kafka Pod
IP = 10.10.1.20
```

Applications using the old IP would fail.

A Service provides a stable endpoint regardless of Pod IP changes.

---

# 2. Metadata

```yaml
metadata:
  name: opentelemetry-demo-kafka
```

## Purpose

This is the Service name.

```text
opentelemetry-demo-kafka
```

Kubernetes automatically creates a DNS record for this Service.

Applications can connect using:

```text
opentelemetry-demo-kafka:9092
```

instead of using Pod IP addresses.

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

* Organize resources
* Identify components
* Select Pods
* Enable monitoring and filtering

### Example

```yaml
app.kubernetes.io/component: kafka
```

Shows that this Service belongs to the Kafka component.

---

# 4. Service Specification

```yaml
spec:
```

Defines how the Kafka Service exposes Kafka Pods.

---

# 5. Service Type

```yaml
type: ClusterIP
```

## What is ClusterIP?

ClusterIP exposes the Service only inside the Kubernetes cluster.

### Access

```text
Inside Cluster  → Allowed
Outside Cluster → Not Allowed
```

### Why ClusterIP?

Kafka is used only by internal microservices.

Example:

```text
Checkout Service
Fraud Detection Service
Payment Services
```

These services communicate with Kafka internally.

There is no requirement to expose Kafka directly to internet users.

---

# 6. Ports Configuration

This Service exposes two ports.

```yaml
ports:
```

---

# 6.1 Kafka Client Port

```yaml
- port: 9092
  name: plaintext
  targetPort: 9092
```

## Purpose

This is the main Kafka communication port.

Applications connect to Kafka using:

```text
opentelemetry-demo-kafka:9092
```

---

### Service Port

```yaml
port: 9092
```

The Service listens on:

```text
9092
```

---

### Target Port

```yaml
targetPort: 9092
```

Traffic is forwarded to port 9092 inside the Kafka container.

---

### Usage

```text
Checkout Service
        |
        v
Kafka Service:9092
        |
        v
Kafka Pod:9092
```

Used by:

* Producers
* Consumers
* Event-driven services

---

# 6.2 Kafka Controller Port

```yaml
- port: 9093
  name: controller
  targetPort: 9093
```

## Purpose

Used for Kafka controller communication.

In Kafka KRaft mode, this port manages:

* Broker coordination
* Metadata management
* Cluster leadership activities

---

### Service Port

```yaml
port: 9093
```

Service listens on:

```text
9093
```

---

### Target Port

```yaml
targetPort: 9093
```

Traffic is forwarded to:

```text
Kafka Pod:9093
```

---

### Internal Flow

```text
Kafka Controller Operations
           |
           v
Kafka Service:9093
           |
           v
Kafka Pod:9093
```

---

# 7. Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-kafka
```

## Purpose

The selector tells Kubernetes which Pods should receive traffic.

Kubernetes searches for Pods having:

```yaml
opentelemetry.io/name: opentelemetry-demo-kafka
```

---

## Matching Example

### Kafka Pod

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-kafka
```

### Service Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-kafka
```

Because both match:

```text
Kafka Service
       |
       v
Kafka Pod
```

Traffic is routed correctly.

---

# Service Discovery

Kubernetes automatically creates a DNS record.

### Service Name

```text
opentelemetry-demo-kafka
```

Applications connect using:

```text
opentelemetry-demo-kafka:9092
```

No Pod IP knowledge is required.

---

# How Applications Use Kafka Service

### Fraud Detection Service

```yaml
KAFKA_SERVICE_ADDR=opentelemetry-demo-kafka:9092
```

When the Fraud Detection Service starts:

```text
Fraud Detection Service
         |
         v
Kafka Service
         |
         v
Kafka Pod
```

The Service handles routing automatically.

---

# Message Flow Example

```text
Checkout Service
        |
        | Publish Order Event
        v
Kafka Service
        |
        v
Kafka Broker
        |
        | Consume Event
        v
Fraud Detection Service
```

---

# What Happens if Kafka Pod Restarts?

### Without Service

```text
Kafka Pod
IP = 10.0.1.5

Pod Restart

New IP = 10.0.1.9

Applications Fail
```

---

### With Service

```text
Kafka Service
        |
        v
Always Routes To
Current Kafka Pod
```

Applications continue working without configuration changes.

---

# Relationship Between Deployment and Service

### Deployment

Creates and manages Kafka Pods.

```text
Kafka Deployment
        |
        v
Kafka Pod
```

### Service

Provides stable access to Kafka.

```text
Kafka Service
        |
        v
Kafka Pod
```

Together:

```text
Deployment Creates Pods
         |
         v
Service Exposes Pods
```

---

# Kafka Communication Architecture

```text
Frontend
    |
    v
Checkout Service
    |
    v
Kafka Service (9092)
    |
    v
Kafka Pod
    |
    v
Fraud Detection Service
```

---

# Complete Traffic Flow

```text
Producer Application
         |
         v
opentelemetry-demo-kafka:9092
         |
         v
Kafka Service
         |
         v
Kafka Pod
         |
         v
Kafka Topic
         |
         v
Consumer Application
```

---

# Summary

This Service:

* Creates a stable network endpoint for Kafka.
* Uses the DNS name `opentelemetry-demo-kafka`.
* Exposes Kafka client traffic on port `9092`.
* Exposes Kafka controller traffic on port `9093`.
* Uses `ClusterIP` for internal cluster communication.
* Automatically discovers Kafka Pods using labels.
* Provides DNS-based service discovery.
* Protects applications from Kafka Pod IP changes.

## In Simple Terms

The Kafka Service acts as a **permanent address for the Kafka broker**. Instead of applications connecting directly to a Kafka Pod whose IP may change, they connect to `opentelemetry-demo-kafka:9092`. Kubernetes automatically forwards the traffic to the running Kafka Pod, ensuring reliable communication between microservices such as Checkout and Fraud Detection.
