# Frontend Proxy Ingress (`ingress.yaml`) Documentation

## Overview

This Kubernetes Ingress resource exposes the **Frontend Proxy Service** to users over the internet using an **AWS Application Load Balancer (ALB)**.

Without Ingress, the Frontend Proxy Service is accessible only inside the Kubernetes cluster because it uses a `ClusterIP` service. The Ingress creates an external entry 
point and routes incoming traffic to the Frontend Proxy service.

```text
Internet User
      |
      v
AWS Application Load Balancer (ALB)
      |
      v
Ingress
      |
      v
Frontend Proxy Service
      |
      v
Frontend Proxy Pod
```

---

# 1. API Version and Resource Type

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
```

## Purpose

### apiVersion

```yaml
networking.k8s.io/v1
```

Uses the Kubernetes Networking API.

### kind

```yaml
Ingress
```

Creates an Ingress resource that manages external HTTP/HTTPS access to services inside the cluster.

---

# 2. Metadata

```yaml
metadata:
  name: frontend-proxy
```

## Purpose

This is the name of the Ingress resource.

```text
frontend-proxy
```

Kubernetes uses this name to identify and manage the Ingress.

---

# 3. AWS ALB Annotations

Annotations provide additional instructions to the AWS Load Balancer Controller.

---

## 3.1 Internet-Facing ALB

```yaml
alb.ingress.kubernetes.io/scheme: internet-facing
```

### Purpose

Creates a public Application Load Balancer.

### Traffic Flow

```text
Internet
    |
    v
Public ALB
    |
    v
Kubernetes Service
```

### Benefit

Users can access the application from anywhere on the internet.

---

## 3.2 Target Type

```yaml
alb.ingress.kubernetes.io/target-type: ip
```

### Purpose

Registers Pod IPs directly with the ALB.

### How It Works

```text
ALB
 |
 +--> Pod IP 1
 +--> Pod IP 2
 +--> Pod IP 3
```

Instead of sending traffic through NodePorts, the ALB sends traffic directly to Pods.

### Benefits

* Better performance
* Simpler networking
* Efficient load balancing

---

# 4. Ingress Specification

```yaml
spec:
```

Defines how incoming traffic should be routed.

---

# 5. Ingress Class

```yaml
ingressClassName: alb
```

## Purpose

Specifies which Ingress Controller should process this Ingress.

### In This Case

```text
AWS Load Balancer Controller
```

The AWS Load Balancer Controller watches for Ingress resources with:

```yaml
ingressClassName: alb
```

and automatically creates an AWS Application Load Balancer.

---

# 6. Routing Rules

```yaml
rules:
```

Rules determine where incoming requests should be sent.

---

## 6.1 Host-Based Routing

```yaml
host: example.com
```

### Purpose

This rule applies only when users access:

```text
http://example.com
```

or

```text
https://example.com
```

### Example

```text
Request: https://example.com
          |
          v
Ingress Rule Matched
```

If another domain is used, this rule will not match.

---

# 7. HTTP Path Configuration

```yaml
http:
  paths:
```

Defines URL paths that should be routed.

---

## 7.1 Root Path

```yaml
path: "/"
pathType: Prefix
```

### Purpose

Matches all URLs beginning with `/`.

### Examples

Matched:

```text
/
```

```text
/checkout
```

```text
/products
```

```text
/cart
```

```text
/api/orders
```

All requests starting with `/` are forwarded to the Frontend Proxy Service.

---

## Path Type

```yaml
pathType: Prefix
```

### Meaning

Any request beginning with the specified path is matched.

Example:

```text
Path = /
```

Matches:

```text
/
```

```text
/anything
```

```text
/products/item1
```

---

# 8. Backend Service

```yaml
backend:
  service:
```

Defines where traffic should be sent.

---

## Service Name

```yaml
name: opentelemetry-demo-frontendproxy
```

### Purpose

Traffic is forwarded to the Frontend Proxy Service.

```text
Ingress
   |
   v
opentelemetry-demo-frontendproxy
```

---

## Service Port

```yaml
port:
  number: 8080
```

### Purpose

The Ingress sends requests to Service Port:

```text
8080
```

Flow:

```text
Internet User
      |
      v
ALB
      |
      v
Ingress
      |
      v
Frontend Proxy Service:8080
      |
      v
Frontend Proxy Pod:8080
```

---

# Complete Request Flow

When a user accesses the application:

```text
https://example.com
        |
        v
AWS ALB
        |
        v
Frontend Proxy Ingress
        |
        v
Frontend Proxy Service
        |
        v
Frontend Proxy Pod
        |
        v
Frontend Application
```

---

# How AWS Creates Resources

After applying this Ingress:

```bash
kubectl apply -f ingress.yaml
```

The AWS Load Balancer Controller automatically:

### Step 1

Creates an AWS Application Load Balancer.

```text
AWS ALB
```

### Step 2

Creates Target Groups.

```text
Frontend Proxy Target Group
```

### Step 3

Registers Frontend Proxy Pod IPs.

```text
Pod IPs
```

### Step 4

Creates Listener Rules.

```text
example.com/*
      |
      v
Frontend Proxy Service
```

---

# Why Ingress is Needed

Without Ingress:

```text
Internet User
      X
Cannot Access Service
```

Because:

```yaml
type: ClusterIP
```

allows only internal cluster communication.

With Ingress:

```text
Internet User
      |
      v
ALB
      |
      v
Frontend Proxy Service
```

The application becomes accessible from outside the cluster.

---

# Summary

This Ingress:

* Creates an external entry point for the application.
* Uses AWS Application Load Balancer (ALB).
* Creates a public internet-facing load balancer.
* Routes traffic for `example.com`.
* Matches all URLs beginning with `/`.
* Forwards requests to `opentelemetry-demo-frontendproxy`.
* Sends traffic to Service Port `8080`.
* Uses Pod IPs directly as ALB targets.
* Makes the Frontend Proxy accessible from the internet.

## In Simple Terms

This Ingress acts as the **main internet gateway** for the OpenTelemetry Demo application. When users open `example.com`, the AWS Application Load Balancer receives the 
request and forwards it through the Ingress to the Frontend Proxy Service running inside the Kubernetes cluster. This is what allows users outside the cluster to access 
the application.
