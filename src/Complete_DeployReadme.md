# Introduction to the Deployment File

## What is this file?

This file is a **Kubernetes Deployment Manifest** that contains all the Kubernetes resources required to deploy the OpenTelemetry Demo application into a Kubernetes cluster.

Instead of creating resources one by one, all resources are defined in a single YAML file and applied together using:

```bash
kubectl apply -f complete-deploy.yaml
```

When Kubernetes reads this file, it creates all required objects such as:

* ServiceAccount
* ConfigMap
* Services
* Deployments
* Pods

These resources work together to run the complete microservices application.

---

## Why do we need this file?

Imagine you have an e-commerce application consisting of multiple microservices:

```text
Frontend
│
├── Product Catalog Service
├── Cart Service
├── Checkout Service
├── Payment Service
├── Shipping Service
├── Recommendation Service
├── Currency Service
├── Ad Service
└── Email Service
```

Each microservice needs:

* A container image
* Network configuration
* Environment variables
* Kubernetes Service
* Deployment configuration

Managing all these manually would be difficult.

Therefore Kubernetes uses YAML manifest files to describe the desired state of the application.

---

## What happens when this file is applied?

When you run:

```bash
kubectl apply -f complete-deploy.yaml
```

Kubernetes performs the following actions:

### Step 1: Creates ServiceAccount

```text
ServiceAccount
     │
     ▼
Provides identity to Pods
```

Pods use this identity when communicating with the Kubernetes API.

---

### Step 2: Creates ConfigMaps

```text
ConfigMap
     │
     ▼
Stores application configuration
```

Instead of hardcoding configuration inside containers, Kubernetes stores it separately.

---

### Step 3: Creates Deployments

```text
Deployment
     │
     ▼
Creates Pods
```

Example:

```text
Deployment
    │
    ▼
Pod
    │
    ▼
Container
```

Deployments ensure required Pods are always running.

---

### Step 4: Creates Services

```text
Service
     │
     ▼
Provides stable networking
```

Pods can die and get recreated.

Their IP addresses change.

Services provide a permanent name such as:

```text
cartservice
paymentservice
frontend
```

so applications can communicate reliably.

# Chapter 2: ServiceAccount

The first resource in your deployment file is:

```yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: opentelemetry-demo
  labels:
    opentelemetry.io/name: opentelemetry-demo
    app.kubernetes.io/instance: opentelemetry-demo
    app.kubernetes.io/name: opentelemetry-demo
    app.kubernetes.io/version: "1.12.0"
    app.kubernetes.io/part-of: opentelemetry-demo
```

Before understanding each line, first understand **why this resource exists**.

---

# What is a ServiceAccount?

A ServiceAccount is an identity for Pods running inside Kubernetes.

Think of it like this:

```text
Human User
    │
    ▼
User Account

Pod
    │
    ▼
ServiceAccount
```

Humans use user accounts.

Pods use ServiceAccounts.

---

# Why Do Pods Need an Identity?

Suppose a pod wants to:

* Read ConfigMaps
* Read Secrets
* Access Kubernetes API
* Discover cluster information

Kubernetes must know:

```text
Who is making this request?
```

The answer is:

```text
This Pod is using ServiceAccount X
```

That ServiceAccount becomes the Pod's identity.

---

# Real-Life Example

Imagine an office building.

Every employee has:

```text
Employee ID Card
```

The card determines:

* Who they are
* What rooms they can access
* What actions they can perform

Similarly:

```text
Pod
 │
 ▼
ServiceAccount
 │
 ▼
Identity inside Kubernetes
```

---

# Why is it Used in This Project?

This OpenTelemetry demo contains many microservices.

Examples:

```text
Frontend
Cart Service
Checkout Service
Payment Service
Product Catalog Service
Recommendation Service
```

All these Pods need an identity.

Instead of creating separate ServiceAccounts for every service, this demo uses:

```yaml
name: opentelemetry-demo
```

and multiple workloads can use it.

---

# YAML Breakdown

Let's examine every line.

---

## YAML Document Separator

```yaml
---
```

### Purpose

Marks the beginning of a new Kubernetes resource.

Without it:

```text
Kubernetes cannot distinguish
where one resource ends
and another begins.
```

Since your file contains 38 resources, every resource starts with:

```yaml
---
```

---

# apiVersion

```yaml
apiVersion: v1
```

## What is apiVersion?

Every Kubernetes object belongs to an API group.

Kubernetes must know:

```text
Which API version understands this resource?
```

For ServiceAccount:

```yaml
apiVersion: v1
```

is the stable core Kubernetes API.

---

### Think of it as:

```text
Kubernetes API
    │
    ├── v1
    ├── apps/v1
    ├── networking.k8s.io/v1
    └── batch/v1
```

Different resources belong to different API groups.

Examples:

```yaml
ServiceAccount -> v1
Service        -> v1
ConfigMap      -> v1
Deployment     -> apps/v1
Job            -> batch/v1
```

---

# kind

```yaml
kind: ServiceAccount
```

## What is kind?

It tells Kubernetes:

```text
What object should be created?
```

In this case:

```yaml
kind: ServiceAccount
```

means:

```text
Create a ServiceAccount object.
```

Examples:

```yaml
kind: Service
kind: Deployment
kind: ConfigMap
kind: Secret
```

Each creates a different Kubernetes resource.

---

# metadata Section

```yaml
metadata:
```

This section contains information about the object itself.

Think of it as:

```text
Object Information
```

Similar to:

```text
Name
Version
Tags
Labels
```

for an application.

---

# Name

```yaml
name: opentelemetry-demo
```

This is the resource name.

Kubernetes stores this ServiceAccount as:

```text
opentelemetry-demo
```

You can verify it:

```bash
kubectl get serviceaccounts
```

Output:

```text
NAME
opentelemetry-demo
```

---

# Labels

Now we reach:

```yaml
labels:
```

Labels are key-value pairs attached to Kubernetes resources.

Think of them as tags.

Example:

```text
Environment = Production
Team = Platform
Application = Payment
```

Kubernetes uses labels for:

* Searching
* Filtering
* Monitoring
* Resource grouping
* Service selection

---

# Label 1

```yaml
opentelemetry.io/name: opentelemetry-demo
```

### Purpose

Custom label created by OpenTelemetry project.

Meaning:

```text
This resource belongs to the
OpenTelemetry Demo application.
```

Useful when:

```bash
kubectl get all -l opentelemetry.io/name=opentelemetry-demo
```

---

# Label 2

```yaml
app.kubernetes.io/instance: opentelemetry-demo
```

### Purpose

Identifies a specific application instance.

Imagine:

```text
Development Environment
Production Environment
Testing Environment
```

All may run the same application.

This label tells Kubernetes:

```text
This resource belongs to the
opentelemetry-demo instance.
```

---

# Label 3

```yaml
app.kubernetes.io/name: opentelemetry-demo
```

### Purpose

Defines application name.

Think:

```text
Application Name
```

Kubernetes tools and dashboards often display this label.

---

# Label 4

```yaml
app.kubernetes.io/version: "1.12.0"
```

### Purpose

Stores application version.

Meaning:

```text
This deployment uses
OpenTelemetry Demo version 1.12.0
```

Benefits:

* Upgrade tracking
* Troubleshooting
* Monitoring

Example:

```bash
kubectl get all --show-labels
```

You can immediately identify versions.

---

# Label 5

```yaml
app.kubernetes.io/part-of: opentelemetry-demo
```

### Purpose

Indicates that this resource belongs to a larger application.

Think:

```text
Frontend
Cart Service
Payment Service
Email Service
```

All belong to:

```text
OpenTelemetry Demo
```

This label creates that relationship.

---

# What Happens After Creation?

When Kubernetes creates this ServiceAccount:

```text
ServiceAccount
      │
      ▼
Stored in etcd
      │
      ▼
Available for Pods
```

Deployments can later reference it:

```yaml
spec:
  serviceAccountName: opentelemetry-demo
```

Meaning:

```text
Run Pods using this identity.
```

---


---

