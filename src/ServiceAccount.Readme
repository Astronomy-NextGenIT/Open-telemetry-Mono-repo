# Kubernetes Manifest Documentation

## ServiceAccount: opentelemetry-demo

---

## 1. OBJECT OVERVIEW

```yaml id="sa0"
apiVersion: v1
kind: ServiceAccount
```

### What this means

This block creates a **ServiceAccount** in Kubernetes.

A ServiceAccount is:

> A Kubernetes identity assigned to pods to control what they can access inside the cluster.

---

## 2. METADATA SECTION

```yaml id="sa1"
metadata:
  name: opentelemetry-demo
```

### Field explanation

### `metadata`

Holds identifying information about the Kubernetes object.

---

### `name: opentelemetry-demo`

* Name of the ServiceAccount
* Used by pods via:

```yaml id="sa2"
serviceAccountName: opentelemetry-demo
```

### Meaning in simple terms

All pods using this will "run as" this identity.

---

## 3. LABELS SECTION (VERY IMPORTANT)

```yaml id="sa3"
labels:
  opentelemetry.io/name: opentelemetry-demo
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/name: opentelemetry-demo
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

---

## WHAT ARE LABELS?

Labels are:

> Key-value metadata used for grouping, filtering, and connecting Kubernetes objects.

They are NOT functional logic, but are used by:

* Services (to find pods)
* Helm (to manage releases)
* Monitoring tools (Prometheus, Grafana)
* Debugging commands (`kubectl get pods -l`)

---

## LABEL-BY-LABEL EXPLANATION

---

### 1. `opentelemetry.io/name`

```text id="l1"
opentelemetry-demo
```

* Custom label for OpenTelemetry ecosystem
* Used for grouping all demo components

---

### 2. `app.kubernetes.io/instance`

```text id="l2"
opentelemetry-demo
```

* Represents a specific deployed instance
* Useful when multiple environments exist (dev, prod)

---

### 3. `app.kubernetes.io/name`

```text id="l3"
opentelemetry-demo
```

* Logical application name
* Stable identifier across deployments

---

### 4. `app.kubernetes.io/version`

```text id="l4"
1.12.0
```

* Version of the application stack
* Important for tracking upgrades/rollbacks

---

### 5. `app.kubernetes.io/part-of`

```text id="l5"
opentelemetry-demo
```

* Groups multiple microservices under one system
* Useful for identifying full application stack

---

## HOW THIS IS USED IN REAL CLUSTER

### 1. Pod Identity

When a pod uses:

```yaml id="p1"
serviceAccountName: opentelemetry-demo
```

It inherits this identity.

---

### 2. RBAC (if configured)

This ServiceAccount can be attached to:

* Roles
* ClusterRoles
* Permissions (API access control)

---

### 3. Observability grouping

Tools like:

* Prometheus
* Grafana
* Jaeger

use labels to group services visually.

---

## RELATIONSHIP MAP

```text id="m1"
ServiceAccount
   ↓
Pods (Deployments)
   ↓
Services (networking)
   ↓
Observability tools (via labels)
```

---

