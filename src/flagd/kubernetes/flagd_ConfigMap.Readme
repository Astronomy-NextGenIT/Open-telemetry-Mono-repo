Since you're creating documentation for every Kubernetes manifest, this file should be documented as a **ConfigMap**, not a Deployment or Service.

# Flagd ConfigMap Documentation

## Overview

This `flagd-config.yaml` file creates a Kubernetes **ConfigMap** that stores feature flag definitions used by the Flagd service.

The ConfigMap contains a JSON configuration file (`demo.flagd.json`) that defines various feature flags used throughout the OpenTelemetry Demo application.

Instead of hardcoding feature behavior into application code, developers can enable or disable features dynamically through these flags.

---

## What is a ConfigMap?

A ConfigMap is a Kubernetes object used to store non-sensitive configuration data.

Without ConfigMap:

```text
Configuration Inside Application
            ↓
Code Change Required
            ↓
Rebuild Image
            ↓
Redeploy Application
```

With ConfigMap:

```text
Configuration Stored Separately
            ↓
Application Reads Configuration
            ↓
Behavior Changes Dynamically
```

---

## Resource Definition

```yaml
apiVersion: v1
kind: ConfigMap
```

### Purpose

Creates a Kubernetes ConfigMap resource.

---

## ConfigMap Name

```yaml
metadata:
  name: opentelemetry-demo-flagd-config
```

### Purpose

Provides a unique identifier for the ConfigMap.

### Used By

```text
Flagd Deployment
      ↓
Reads ConfigMap
      ↓
Loads Feature Flags
```

---

## Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/name: opentelemetry-demo
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

### Purpose

Labels help Kubernetes:

* Identify resources
* Organize application components
* Support monitoring
* Simplify troubleshooting

---

# ConfigMap Data

The ConfigMap contains a file called:

```yaml
demo.flagd.json
```

This file contains all feature flag definitions.

---

# Configuration File Structure

```text
ConfigMap
      ↓
demo.flagd.json
      ↓
Feature Flags
      ↓
Flagd Service
      ↓
Applications
```

---

# JSON Schema

```json
"$schema": "https://flagd.dev/schema/v0/flags.json"
```

### Purpose

Defines the Flagd schema used to validate feature flag configurations.

---

# Understanding Feature Flags

Each feature flag contains:

```json
{
  "description": "What the flag does",
  "state": "ENABLED",
  "variants": {
    "on": true,
    "off": false
  },
  "defaultVariant": "off"
}
```

### Components

| Field          | Purpose                     |
| -------------- | --------------------------- |
| description    | Explains the flag           |
| state          | Enables the flag definition |
| variants       | Possible values             |
| defaultVariant | Default behavior            |

---

# Feature Flag: Product Catalog Failure

```json
productCatalogFailure
```

### Purpose

Simulates failures in the Product Catalog Service.

### When Enabled

```text
Product Catalog Requests
          ↓
Artificial Failure
          ↓
Error Generated
```

### Use Case

Testing application resiliency and error handling.

---

# Feature Flag: Recommendation Service Cache Failure

```json
recommendationServiceCacheFailure
```

### Purpose

Simulates cache failures in Recommendation Service.

### When Enabled

```text
Cache Access
      ↓
Failure Generated
      ↓
Fallback Logic Tested
```

---

# Feature Flag: Ad Service Manual Garbage Collection

```json
adServiceManualGc
```

### Purpose

Triggers manual garbage collection in Ad Service.

### When Enabled

```text
Ad Service
      ↓
Manual GC Triggered
      ↓
Performance Impact Observed
```

### Use Case

Testing memory management behavior.

---

# Feature Flag: Ad Service High CPU

```json
adServiceHighCpu
```

### Purpose

Creates high CPU usage inside Ad Service.

### When Enabled

```text
Ad Service
      ↓
CPU Load Increased
      ↓
Performance Monitoring Tested
```

### Use Case

Observability demonstrations.

---

# Feature Flag: Ad Service Failure

```json
adServiceFailure
```

### Purpose

Simulates complete Ad Service failures.

### When Enabled

```text
Ad Request
      ↓
Service Failure
      ↓
Error Returned
```

---

# Feature Flag: Kafka Queue Problems

```json
kafkaQueueProblems
```

### Purpose

Simulates Kafka bottlenecks and consumer lag.

### Variants

```json
"on": 100
"off": 0
```

### When Enabled

```text
Kafka Queue
      ↓
Large Message Backlog
      ↓
Consumer Delay
      ↓
Lag Spike
```

### Use Case

Testing event-driven systems.

---

# Feature Flag: Cart Service Failure

```json
cartServiceFailure
```

### Purpose

Simulates Cart Service failures.

### When Enabled

```text
Add To Cart Request
         ↓
Cart Service Failure
         ↓
Error Response
```

---

# Feature Flag: Payment Service Failure

```json
paymentServiceFailure
```

### Purpose

Simulates payment processing failures.

### When Enabled

```text
Payment Request
      ↓
Payment Rejected
      ↓
Checkout Failure
```

### Use Case

Testing checkout error handling.

---

# Feature Flag: Payment Service Unreachable

```json
paymentServiceUnreachable
```

### Purpose

Simulates a completely unavailable Payment Service.

### When Enabled

```text
Checkout Service
      ↓
Payment Service
      ↓
Connection Failure
```

### Use Case

Testing network outage scenarios.

---

# Feature Flag: Load Generator Homepage Flood

```json
loadgeneratorFloodHomepage
```

### Purpose

Generates heavy traffic against the Frontend.

### Variants

```json
"on": 100
"off": 0
```

### When Enabled

```text
Load Generator
      ↓
Large Number Of Requests
      ↓
Homepage Traffic Spike
```

### Use Case

Load testing demonstrations.

---

# Feature Flag: Image Slow Load

```json
imageSlowLoad
```

### Purpose

Simulates slow-loading images.

### Variants

```json
"10sec": 10000
"5sec": 5000
"off": 0
```

### Behavior

```text
10sec → 10,000ms Delay
5sec  → 5,000ms Delay
off   → No Delay
```

### Use Case

Testing frontend performance monitoring.

---

# How ConfigMap Is Used By Flagd

From the Flagd Deployment:

```text
ConfigMap
      ↓
config-ro Volume
      ↓
Init Container
      ↓
config-rw Volume
      ↓
Flagd Container
      ↓
Load Feature Flags
```

---

# Complete Startup Flow

```text
kubectl apply flagd-config.yaml
            ↓
ConfigMap Created
            ↓
Feature Flag Definitions Stored
            ↓

kubectl apply deployment.yaml
            ↓
Flagd Pod Created
            ↓
ConfigMap Mounted
            ↓
Init Container Copies File
            ↓
Flagd Reads demo.flagd.json
            ↓
Feature Flags Loaded
            ↓
Applications Can Query Flags
```

---

# Runtime Feature Flag Flow

```text
Application
      ↓
Request Feature Status
      ↓
Flagd Service
      ↓
Read ConfigMap Data
      ↓
Find Matching Flag
      ↓
Return Enabled/Disabled Value
      ↓
Application Changes Behavior
```

---

# Why This ConfigMap Is Important

Without this ConfigMap:

```text
Feature Behavior
        ↓
Hardcoded In Applications
        ↓
Code Changes Required
```

With this ConfigMap:

```text
Feature Definitions
        ↓
Stored Centrally
        ↓
Managed By Flagd
        ↓
Dynamic Feature Control
```

---

# Summary

This ConfigMap file:

* Stores all Flagd feature flag definitions
* Creates the `demo.flagd.json` configuration file
* Provides centralized feature management
* Supports failure simulation scenarios
* Supports performance testing scenarios
* Supports load testing scenarios
* Allows dynamic feature control without code changes
* Is mounted into the Flagd Deployment
* Enables observability and resiliency demonstrations
* Acts as the source of truth for all application feature flags

The overall goal of this ConfigMap is to provide centralized configuration for Flagd, enabling dynamic control of application behavior, failure simulations, performance testing, and observability demonstrations without requiring application redeployments.
