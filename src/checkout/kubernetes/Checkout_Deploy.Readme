# Checkout Service Deployment Documentation

## Overview

This `deploy.yaml` file is responsible for deploying the **Checkout Service** in the Kubernetes cluster.

The Checkout Service acts as the central coordinator of the e-commerce application. When a customer places an order, this service communicates with multiple 
backend services to complete the checkout process successfully.

The Checkout Service interacts with:

* Cart Service
* Product Catalog Service
* Payment Service
* Shipping Service
* Currency Service
* Email Service
* Kafka
* Flagd
* OpenTelemetry Collector

This Deployment ensures that the Checkout Service is always running, monitored, restarted automatically if it fails, and properly connected to all required dependencies.

---

# High-Level Architecture

```text
Customer
    ↓
Frontend
    ↓
Checkout Service
    ↓
 ┌────────────┬────────────┬────────────┬────────────┐
 ↓            ↓            ↓            ↓
Cart      Payment      Shipping     Product Catalog
Service   Service      Service      Service
 ↓            ↓            ↓
Currency    Email      Kafka
Service     Service
```

---

# Deployment Resource

```yaml
apiVersion: apps/v1
kind: Deployment
```

## Purpose

Defines a Kubernetes Deployment resource.

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
Checkout Service Started
```

The Deployment continuously manages the application's lifecycle.

---

# Deployment Name

```yaml
metadata:
  name: opentelemetry-demo-checkoutservice
```

## Purpose

Provides a unique name for the Deployment resource.

### Usage

Kubernetes uses this name to:

* Identify the Deployment
* Manage updates
* Perform rollbacks
* Monitor status

---

# Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-checkoutservice
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: checkoutservice
  app.kubernetes.io/name: opentelemetry-demo-checkoutservice
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

## Purpose

Labels provide metadata about the application.

### Benefits

Labels help Kubernetes:

* Identify resources
* Group components
* Match Services with Pods
* Monitor applications
* Filter resources

Example:

```yaml
app.kubernetes.io/component: checkoutservice
```

identifies this Pod as the Checkout Service component.

---

# Replica Configuration

```yaml
replicas: 1
```

## Purpose

Defines how many Pod instances should run.

### Current Configuration

```text
1 Checkout Service Pod
```

### Failure Recovery

```text
Pod Fails
    ↓
Deployment Detects Failure
    ↓
New Pod Created
```

Kubernetes automatically maintains the desired state.

---

# Revision History

```yaml
revisionHistoryLimit: 10
```

## Purpose

Stores up to 10 previous Deployment versions.

### Benefit

Supports rollback if an update causes problems.

```text
Version 1.12.0
      ↓
Version 1.13.0
      ↓
Issue Detected
      ↓
Rollback Possible
```

---

# Selector

```yaml
selector:
  matchLabels:
    opentelemetry.io/name: opentelemetry-demo-checkoutservice
```

## Purpose

Defines which Pods belong to this Deployment.

### How It Works

```text
Deployment
      ↓
Find Pods With Matching Label
      ↓
Manage Those Pods
```

---

# Pod Template

```yaml
template:
```

## Purpose

Acts as the blueprint for creating Pods.

Whenever a Pod needs to be created, Kubernetes uses this template.

---

# Service Account

```yaml
serviceAccountName: opentelemetry-demo
```

## Purpose

Associates the Pod with a Kubernetes Service Account.

### Benefit

Provides controlled permissions for Kubernetes operations.

---

# Main Application Container

```yaml
containers:
  - name: checkoutservice
```

## Purpose

Defines the main Checkout Service container.

This container executes the business logic of the checkout workflow.

---

# Container Image

```yaml
image: ghcr.io/open-telemetry/demo:1.12.0-checkoutservice
```

## Purpose

Specifies the Docker image used to run the application.

### Image Source

```text
GitHub Container Registry (GHCR)
```

### Startup Flow

```text
Node
 ↓
Pull Image
 ↓
Create Container
 ↓
Start Checkout Service
```

---

# Image Pull Policy

```yaml
imagePullPolicy: IfNotPresent
```

## Purpose

Controls image download behavior.

### Behavior

```text
Image Exists Locally
        ↓
Use Existing Image

Image Missing
        ↓
Download From Registry
```

This reduces deployment time.

---

# Container Port

```yaml
ports:
  - containerPort: 8080
    name: service
```

## Purpose

Exposes port 8080 inside the container.

### What Happens

The Checkout Service listens on:

```text
Port 8080
```

for incoming requests.

---

# Environment Variables

Environment variables provide runtime configuration to the application.

---

## OTEL_SERVICE_NAME

```yaml
OTEL_SERVICE_NAME
```

### Purpose

Automatically retrieves the service name from Pod labels.

### Result

```text
checkoutservice
```

This name appears in traces, metrics, and logs.

---

## OTEL_COLLECTOR_NAME

```yaml
OTEL_COLLECTOR_NAME=opentelemetry-demo-otelcol
```

### Purpose

Defines the OpenTelemetry Collector service.

---

## Metrics Temporality

```yaml
OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE=cumulative
```

### Purpose

Controls how metrics are reported.

### Meaning

```text
Metrics continue accumulating over time.
```

This is useful for long-term monitoring.

---

## CHECKOUT_SERVICE_PORT

```yaml
CHECKOUT_SERVICE_PORT=8080
```

### Purpose

Defines the application listening port.

---

# Cart Service Integration

```yaml
CART_SERVICE_ADDR=opentelemetry-demo-cartservice:8080
```

## Purpose

Retrieves products stored in a user's shopping cart.

### Flow

```text
Checkout Service
      ↓
Cart Service
      ↓
Cart Information
```

---

# Currency Service Integration

```yaml
CURRENCY_SERVICE_ADDR=opentelemetry-demo-currencyservice:8080
```

## Purpose

Converts prices between currencies.

### Flow

```text
Checkout Service
      ↓
Currency Service
      ↓
Price Conversion
```

---

# Email Service Integration

```yaml
EMAIL_SERVICE_ADDR=http://opentelemetry-demo-emailservice:8080
```

## Purpose

Sends order confirmation emails.

### Flow

```text
Order Completed
      ↓
Checkout Service
      ↓
Email Service
      ↓
Customer Receives Email
```

---

# Payment Service Integration

```yaml
PAYMENT_SERVICE_ADDR=opentelemetry-demo-paymentservice:8080
```

## Purpose

Processes customer payments.

### Flow

```text
Checkout Service
      ↓
Payment Service
      ↓
Payment Approved
```

---

# Product Catalog Service Integration

```yaml
PRODUCT_CATALOG_SERVICE_ADDR=opentelemetry-demo-productcatalogservice:8080
```

## Purpose

Retrieves product information.

### Flow

```text
Checkout Service
      ↓
Product Catalog Service
      ↓
Product Details
```

---

# Shipping Service Integration

```yaml
SHIPPING_SERVICE_ADDR=opentelemetry-demo-shippingservice:8080
```

## Purpose

Calculates shipping information and delivery costs.

### Flow

```text
Checkout Service
      ↓
Shipping Service
      ↓
Shipping Cost
```

---

# Kafka Integration

```yaml
KAFKA_SERVICE_ADDR=opentelemetry-demo-kafka:9092
```

## Purpose

Publishes checkout-related events.

### Examples

```text
Order Created
Payment Successful
Shipping Started
```

### Flow

```text
Checkout Service
      ↓
Kafka
      ↓
Event Distribution
```

---

# Feature Flag Integration

```yaml
FLAGD_HOST=opentelemetry-demo-flagd
FLAGD_PORT=8013
```

## Purpose

Connects to Flagd feature management service.

### Benefits

```text
Enable Features
Disable Features
A/B Testing
Gradual Rollouts
```

without redeploying the application.

---

# OpenTelemetry Endpoint

```yaml
OTEL_EXPORTER_OTLP_ENDPOINT=http://opentelemetry-demo-otelcol:4317
```

## Purpose

Defines where telemetry data should be exported.

### Data Sent

```text
Traces
Metrics
Logs
```

---

# Resource Attributes

```yaml
OTEL_RESOURCE_ATTRIBUTES
```

## Purpose

Adds metadata to telemetry records.

### Example

```text
service.name=checkoutservice
service.namespace=opentelemetry-demo
service.version=1.12.0
```

This helps identify the service in observability tools.

---

# Resource Limits

```yaml
resources:
  limits:
    memory: 20Mi
```

## Purpose

Restricts memory usage.

### Maximum Allowed

```text
20 MiB
```

This prevents excessive resource consumption.

---

# Init Container

```yaml
initContainers:
```

## Purpose

Runs before the main Checkout Service container starts.

---

# Wait For Kafka

```yaml
until nc -z -v -w30 opentelemetry-demo-kafka 9092
```

## Purpose

Verifies that Kafka is available before starting the application.

### Why Needed

Checkout Service depends on Kafka.

Without this check:

```text
Checkout Service Starts
         ↓
Kafka Not Available
         ↓
Connection Errors
```

With init container:

```text
Check Kafka
      ↓
Kafka Ready?
      ↓
Yes
      ↓
Start Checkout Service
```

---

# Volumes and Volume Mounts

```yaml
volumeMounts:

volumes:
```

## Purpose

Used for attaching storage to containers.

### Current State

No storage is currently configured.

These sections are reserved for future use such as:

* Persistent Volumes
* Config Files
* Secrets
* Shared Storage

---

# Complete Checkout Workflow

```text
Customer Clicks Checkout
            ↓
Checkout Service
            ↓
Retrieve Cart Information
            ↓
Retrieve Product Details
            ↓
Convert Currency
            ↓
Calculate Shipping
            ↓
Process Payment
            ↓
Publish Event To Kafka
            ↓
Send Confirmation Email
            ↓
Order Completed Successfully
```

---

# Complete Deployment Flow

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
Wait For Kafka
            ↓
Kafka Available
            ↓
Checkout Service Container Starts
            ↓
Port 8080 Opened
            ↓
Dependencies Connected
            ↓
Telemetry Enabled
            ↓
Checkout Service Ready
```

---

# Summary

This Deployment file:

* Deploys the Checkout Service application
* Maintains one running Pod
* Exposes application port 8080
* Connects to Cart Service
* Connects to Currency Service
* Connects to Email Service
* Connects to Payment Service
* Connects to Product Catalog Service
* Connects to Shipping Service
* Connects to Kafka
* Connects to Flagd
* Sends telemetry to OpenTelemetry Collector
* Waits for Kafka before startup
* Automatically recreates failed Pods
* Supports rolling updates and rollbacks
* Restricts memory usage to 20Mi

The Checkout Service acts as the central orchestrator of the order placement process, coordinating multiple microservices to complete a customer's purchase while generating telemetry data for observability and monitoring.
