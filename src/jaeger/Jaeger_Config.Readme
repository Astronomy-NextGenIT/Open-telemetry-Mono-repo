# Jaeger OpenTelemetry Collector Configuration (`jaeger-config.yaml`) Documentation

## Overview

This configuration file defines how the **Jaeger service** receives, stores, and displays distributed traces in the OpenTelemetry Demo application.

Jaeger is used for **distributed tracing**, which helps developers track requests as they travel across multiple microservices.

### Example

When a user places an order:

```text
User Request
     |
     v
Frontend
     |
     v
Cart Service
     |
     v
Checkout Service
     |
     v
Payment Service
     |
     v
Shipping Service
```

Jaeger captures the entire journey and displays it as a trace.

---

# Architecture Overview

```text
Application Services
         |
         v
OpenTelemetry Collector
         |
         v
Jaeger Receiver (OTLP)
         |
         v
Memory Storage
         |
         v
Jaeger UI
         |
         v
Developer
```

---

# 1. Service Section

```yaml
service:
```

The `service` section defines:

* Extensions
* Pipelines
* Internal telemetry

---

# 2. Extensions

```yaml
extensions:
  [jaeger_storage, jaeger_query, healthcheckv2]
```

## Purpose

Extensions add additional functionality to Jaeger.

### Loaded Extensions

| Extension      | Purpose               |
| -------------- | --------------------- |
| healthcheckv2  | Health monitoring     |
| jaeger_storage | Trace storage backend |
| jaeger_query   | Trace search and UI   |

---

# 3. Trace Pipeline

```yaml
pipelines:
  traces:
```

Defines how traces move through the system.

---

## Trace Flow

```yaml
receivers: [otlp]
processors: [batch]
exporters: [jaeger_storage_exporter]
```

### Flow Diagram

```text
Incoming Traces
       |
       v
OTLP Receiver
       |
       v
Batch Processor
       |
       v
Storage Exporter
       |
       v
Memory Backend
```

---

# 4. OTLP Receiver

```yaml
receivers:
  otlp:
```

## Purpose

Receives telemetry data from applications.

---

## Supported Protocol

```yaml
grpc:
```

Endpoint:

```yaml
endpoint: ${env:JAEGER_HOST}:${env:JAEGER_GRPC_PORT}
```

### Example

If environment variables are:

```text
JAEGER_HOST=0.0.0.0
JAEGER_GRPC_PORT=4317
```

Result:

```text
0.0.0.0:4317
```

### Purpose

Applications send traces to Jaeger using OTLP over gRPC.

---

# 5. Batch Processor

```yaml
processors:
  batch:
```

## Purpose

Groups traces before writing them to storage.

### Without Batch

```text
Trace 1 -> Store
Trace 2 -> Store
Trace 3 -> Store
```

Many storage operations.

---

### With Batch

```text
Trace 1
Trace 2
Trace 3
   |
   v
Batch
   |
   v
Store Together
```

### Benefits

* Better performance
* Lower resource usage
* Reduced storage operations

---

# 6. Jaeger Storage Exporter

```yaml
exporters:
  jaeger_storage_exporter:
```

## Purpose

Stores traces in the configured backend.

---

## Storage Backend

```yaml
trace_storage: memory_backend
```

### Meaning

All traces are stored in memory (RAM).

---

### Flow

```text
Received Trace
      |
      v
Memory Backend
      |
      v
Jaeger UI
```

---

# 7. Service Telemetry

```yaml
telemetry:
```

Defines monitoring for the Jaeger service itself.

---

# 8. Resource Information

```yaml
resource:
  service.name: jaeger
```

## Purpose

Identifies this service as:

```text
jaeger
```

in telemetry systems.

---

# 9. Metrics Configuration

```yaml
metrics:
```

Configures how Jaeger exports its own metrics.

---

## Metrics Level

```yaml
level: detailed
```

### Purpose

Collects detailed operational metrics.

Examples:

* Trace count
* Request count
* Memory usage
* Processing latency

---

# 10. Metrics Export Interval

```yaml
interval: 10000
```

### Meaning

Every:

```text
10 Seconds
```

Jaeger sends metrics.

---

## Timeout

```yaml
timeout: 5000
```

### Meaning

Waits:

```text
5 Seconds
```

for metrics export to complete.

---

# 11. Metrics Exporter

```yaml
otlp:
```

Metrics are exported using OTLP.

---

## Protocol

```yaml
protocol: http/protobuf
```

### Purpose

Uses OTLP over HTTP.

---

## Endpoint

```yaml
endpoint: http://${env:OTEL_COLLECTOR_HOST}:${env:OTEL_COLLECTOR_PORT_HTTP}
```

### Example

```text
http://otel-collector:4318
```

### Purpose

Sends Jaeger metrics to the OpenTelemetry Collector.

---

# 12. Logging Configuration

```yaml
logs:
  level: info
```

## Purpose

Controls log verbosity.

### INFO Level Logs

Examples:

```text
Service Started
Trace Received
Metrics Exported
```

### Benefit

Provides useful operational information without excessive logging.

---

# 13. Health Check Extension

```yaml
healthcheckv2:
```

## Purpose

Provides health monitoring endpoints.

---

## Enable V2 Health Check

```yaml
use_v2: true
```

Uses the newer health check implementation.

---

## Health Endpoint

```yaml
endpoint: 0.0.0.0:13133
```

### Purpose

Health status available on:

```text
http://<jaeger-host>:13133
```

---

### Example Response

```text
Healthy
```

Used by Kubernetes probes and monitoring tools.

---

# 14. Jaeger Query Extension

```yaml
jaeger_query:
```

## Purpose

Provides the Jaeger web UI.

---

## Storage Configuration

```yaml
storage:
```

Defines where traces are read from.

---

### Trace Storage

```yaml
traces: memory_backend
```

Jaeger UI retrieves traces from memory storage.

---

### Metrics Storage

```yaml
metrics: metrics_backend
```

Metrics are retrieved from Prometheus.

---

## Base Path

```yaml
base_path: /jaeger/ui
```

### Purpose

Jaeger UI is accessible at:

```text
http://<host>/jaeger/ui
```

---

# 15. Jaeger Storage Extension

```yaml
jaeger_storage:
```

Defines trace and metrics storage backends.

---

# 16. Memory Trace Storage

```yaml
memory_backend:
```

Uses RAM to store traces.

---

## Maximum Traces

```yaml
max_traces: ${env:MEMORY_MAX_TRACES}
```

### Example

```text
MEMORY_MAX_TRACES=5000
```

Jaeger stores a maximum of:

```text
5000 traces
```

When the limit is reached, older traces are removed.

---

# 17. Metrics Backend

```yaml
metrics_backend:
```

Stores metrics information.

---

## Prometheus Backend

```yaml
prometheus:
```

Uses Prometheus as the metrics source.

---

## Endpoint

```yaml
endpoint: "http://${env:PROMETHEUS_ADDR}"
```

### Example

```text
http://prometheus:9090
```

Jaeger queries Prometheus for metrics.

---

## Normalize Calls

```yaml
normalize_calls: true
```

### Purpose

Standardizes service call metrics.

---

## Normalize Duration

```yaml
normalize_duration: true
```

### Purpose

Standardizes latency metrics.

---

# Complete Trace Lifecycle

```text
Frontend
     |
     v
OTEL Collector
     |
     v
Jaeger OTLP Receiver
     |
     v
Batch Processor
     |
     v
Memory Backend
     |
     v
Jaeger Query Service
     |
     v
Jaeger UI
     |
     v
Developer Views Trace
```

---

# Example User Request Flow

```text
User Places Order
        |
        v
Frontend
        |
        v
Cart Service
        |
        v
Checkout Service
        |
        v
Payment Service
        |
        v
Shipping Service
```

Jaeger records:

```text
Trace ID: abc123

Frontend -> 50ms
Cart -> 20ms
Checkout -> 150ms
Payment -> 300ms
Shipping -> 40ms
```

Developers can see exactly where delays occur.

---

# Summary

This configuration:

* Receives traces using OTLP gRPC.
* Batches traces before storage.
* Stores traces in memory.
* Exposes the Jaeger UI at `/jaeger/ui`.
* Provides health checks on port `13133`.
* Sends Jaeger metrics to the OpenTelemetry Collector.
* Uses Prometheus as the metrics backend.
* Limits stored traces using `MEMORY_MAX_TRACES`.
* Enables detailed observability of microservice requests.

## In Simple Terms

This file configures **Jaeger as the tracing system** for the OpenTelemetry Demo. It receives traces from applications, temporarily stores them in memory, displays them in the Jaeger UI, exposes health endpoints for monitoring, and collects its own metrics through Prometheus and OpenTelemetry. It allows developers to see how requests move across all microservices and quickly identify performance bottlenecks or failures.
