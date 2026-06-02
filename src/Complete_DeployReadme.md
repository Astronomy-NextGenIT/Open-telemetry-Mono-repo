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

# Chapter 3: Service – opentelemetry-demo-adservice

After creating the ServiceAccount, the next resource is:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: opentelemetry-demo-adservice
  labels:
    opentelemetry.io/name: opentelemetry-demo-adservice
    app.kubernetes.io/instance: opentelemetry-demo
    app.kubernetes.io/component: adservice
    app.kubernetes.io/name: opentelemetry-demo-adservice
    app.kubernetes.io/version: "1.12.0"
    app.kubernetes.io/part-of: opentelemetry-demo
spec:
  type: ClusterIP
  ports:
    - port: 8080
      name: tcp-service
      targetPort: 8080
  selector:
    opentelemetry.io/name: opentelemetry-demo-adservice
```

---

# What is a Service?

A Service provides a **stable network endpoint** for Pods.

Think about this situation:

```text
Deployment
    │
    ▼
Pod-1
```

If Pod-1 crashes:

```text
Pod-1 ❌
```

Kubernetes creates:

```text
Pod-2
```

The problem:

```text
Pod-1 IP = 10.1.1.5
Pod-2 IP = 10.1.1.9
```

The IP changes.

If other applications communicate directly with Pod IPs, communication breaks.

To solve this problem Kubernetes provides:

```text
Service
```

which gives a permanent name.

---

# Why Does AdService Need a Service?

The AdService is one of the microservices in the OpenTelemetry Demo application.

Other services need to communicate with it.

Instead of calling:

```text
10.1.1.5
```

they call:

```text
opentelemetry-demo-adservice
```

Kubernetes automatically routes traffic to the correct Pod.

---

# Service Flow

```text
Frontend
    │
    ▼
Service
(opentelemetry-demo-adservice)
    │
    ▼
AdService Pod
```

The frontend never directly talks to Pod IPs.

It talks to the Service.

---

# YAML Breakdown

---

## apiVersion

```yaml
apiVersion: v1
```

### Purpose

Specifies which Kubernetes API version should process this resource.

For Services:

```yaml
v1
```

is the stable core API.

---

## kind

```yaml
kind: Service
```

### Purpose

Tells Kubernetes:

```text
Create a Service object.
```

Not a Deployment.

Not a Pod.

Not a ConfigMap.

Specifically a Service.

---

# metadata Section

```yaml
metadata:
```

Contains information about the Service itself.

---

## Service Name

```yaml
name: opentelemetry-demo-adservice
```

This becomes the Service's DNS name.

Applications can use:

```text
opentelemetry-demo-adservice
```

to reach this service.

Example:

```text
http://opentelemetry-demo-adservice:8080
```

---

# Labels

Labels are metadata tags attached to resources.

---

## Label 1

```yaml
opentelemetry.io/name: opentelemetry-demo-adservice
```

Identifies the resource as part of the OpenTelemetry Demo.

Useful for searching resources.

Example:

```bash
kubectl get all -l opentelemetry.io/name=opentelemetry-demo-adservice
```

---

## Label 2

```yaml
app.kubernetes.io/instance: opentelemetry-demo
```

Indicates this resource belongs to the OpenTelemetry Demo deployment instance.

Useful when multiple copies of an application exist.

Example:

```text
opentelemetry-demo-dev
opentelemetry-demo-test
opentelemetry-demo-prod
```

---

## Label 3

```yaml
app.kubernetes.io/component: adservice
```

Very important label.

It identifies:

```text
Which microservice is this?
```

Answer:

```text
adservice
```

This helps monitoring and troubleshooting.

---

## Label 4

```yaml
app.kubernetes.io/name: opentelemetry-demo-adservice
```

Application name label.

Used by Kubernetes dashboards, monitoring systems, and Helm charts.

---

## Label 5

```yaml
app.kubernetes.io/version: "1.12.0"
```

Specifies application version.

Useful during:

* Upgrades
* Rollbacks
* Troubleshooting

---

## Label 6

```yaml
app.kubernetes.io/part-of: opentelemetry-demo
```

Shows that AdService belongs to the larger OpenTelemetry Demo application.

---

# spec Section

```yaml
spec:
```

Defines how the Service should behave.

Think of metadata as:

```text
Who am I?
```

and spec as:

```text
What should I do?
```

---

# Service Type

```yaml
type: ClusterIP
```

This is one of the most important lines.

---

## What is ClusterIP?

ClusterIP means:

```text
Accessible ONLY inside the Kubernetes cluster
```

Not accessible from the internet.

Not accessible from your laptop.

Only other Pods can access it.

---

## Communication Flow

```text
Frontend Pod
      │
      ▼
AdService Service
      │
      ▼
AdService Pod
```

Everything happens inside the cluster.

---

## Why ClusterIP?

Because AdService is an internal microservice.

Users never directly access it.

Only other services need it.

---

## Other Service Types

### ClusterIP

```text
Internal only
```

### NodePort

```text
External access via node port
```

Example:

```text
NodeIP:30080
```

### LoadBalancer

```text
Cloud load balancer
```

Example:

```text
AWS ELB
Azure Load Balancer
GCP Load Balancer
```

Your AdService uses:

```yaml
type: ClusterIP
```

because it is an internal service.

---

# Ports Section

```yaml
ports:
```

Defines network ports.

---

## Port Definition

```yaml
- port: 8080
```

This is the Service port.

Clients connect using:

```text
opentelemetry-demo-adservice:8080
```

---

## Name

```yaml
name: tcp-service
```

Human-readable name for the port.

Useful when:

* Multiple ports exist
* Service meshes are used
* Monitoring tools inspect ports

---

## targetPort

```yaml
targetPort: 8080
```

The port on the container receiving traffic.

---

# Traffic Flow

```text
Client Request
      │
      ▼
Service Port 8080
      │
      ▼
Target Port 8080
      │
      ▼
AdService Container
```

---

# Selector Section

```yaml
selector:
```

This is the most important part of a Service.

---

## Selector Value

```yaml
opentelemetry.io/name: opentelemetry-demo-adservice
```

This tells Kubernetes:

```text
Find Pods having this label
and send traffic to them.
```

---

# Chapter 4: Service – opentelemetry-demo-cartservice

The next resource in the deployment file is:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: opentelemetry-demo-cartservice
  labels:
    opentelemetry.io/name: opentelemetry-demo-cartservice
    app.kubernetes.io/instance: opentelemetry-demo
    app.kubernetes.io/component: cartservice
    app.kubernetes.io/name: opentelemetry-demo-cartservice
    app.kubernetes.io/version: "1.12.0"
    app.kubernetes.io/part-of: opentelemetry-demo
spec:
  type: ClusterIP
  ports:
    - port: 8080
      name: tcp-service
      targetPort: 8080
  selector:
    opentelemetry.io/name: opentelemetry-demo-cartservice
```

---

# What is CartService?

CartService is responsible for managing a customer's shopping cart.

When a user:

* Adds a product to the cart
* Removes a product
* Updates quantity
* Views cart contents

those requests are handled by CartService.

Example:

```text
User
 │
 ▼
Frontend
 │
 ▼
CartService
 │
 ├── Add Item
 ├── Remove Item
 ├── Update Quantity
 └── Get Cart
```

Without CartService, users would not be able to maintain their shopping cart before checkout.

---

# Why Does CartService Need a Service?

CartService runs inside Pods.

Pods are temporary.

Example:

```text
Cart Pod
IP = 10.1.5.10
```

If the Pod crashes:

```text
Old Pod ❌
New Pod Created
IP = 10.1.5.21
```

The IP changes.

Other services cannot depend on changing IPs.

Therefore Kubernetes creates:

```text
opentelemetry-demo-cartservice
```

as a stable network endpoint.

---

# Service Communication Flow

```text
Frontend
    │
    ▼
Cart Service
(opentelemetry-demo-cartservice)
    │
    ▼
Cart Pod
```

Frontend never talks directly to Pod IPs.

It always talks through the Service.

---

# YAML Breakdown

---

## Source Comment

```yaml
# Source: opentelemetry-demo/templates/component.yaml
```

### Purpose

This line is only a comment.

It is ignored by Kubernetes.

It tells us:

```text
This resource was generated
from component.yaml Helm template.
```

Useful for developers when troubleshooting Helm charts.

---

# apiVersion

```yaml
apiVersion: v1
```

### Meaning

Use Kubernetes Core API Version 1.

Services belong to the Core API group.

Examples of other API versions:

```yaml
Deployment -> apps/v1
Job -> batch/v1
Ingress -> networking.k8s.io/v1
```

For Service:

```yaml
apiVersion: v1
```

is correct.

---

# kind

```yaml
kind: Service
```

### Meaning

Create a Kubernetes Service object.

Not a Deployment.

Not a Pod.

Not a ConfigMap.

Specifically a networking Service.

---

# metadata Section

```yaml
metadata:
```

Contains identifying information about this Service.

---

# Service Name

```yaml
name: opentelemetry-demo-cartservice
```

This is the most important metadata field.

Kubernetes creates a DNS record:

```text
opentelemetry-demo-cartservice
```

Other applications can access CartService using:

```text
http://opentelemetry-demo-cartservice:8080
```

instead of remembering Pod IPs.

---

# Labels Section

Labels help Kubernetes organize resources.

Think of them as tags.

---

## Label 1

```yaml
opentelemetry.io/name: opentelemetry-demo-cartservice
```

Identifies this resource as the CartService component of the OpenTelemetry Demo application.

Useful for:

```bash
kubectl get all -l opentelemetry.io/name=opentelemetry-demo-cartservice
```

---

## Label 2

```yaml
app.kubernetes.io/instance: opentelemetry-demo
```

Indicates which application instance owns this resource.

Example:

```text
opentelemetry-demo-dev
opentelemetry-demo-test
opentelemetry-demo-prod
```

This label helps distinguish them.

---

## Label 3

```yaml
app.kubernetes.io/component: cartservice
```

Very important label.

It identifies:

```text
Which microservice is this?
```

Answer:

```text
cartservice
```

Monitoring systems often use this label.

---

## Label 4

```yaml
app.kubernetes.io/name: opentelemetry-demo-cartservice
```

Application name label.

Provides a standard Kubernetes naming convention.

---

## Label 5

```yaml
app.kubernetes.io/version: "1.12.0"
```

Indicates the application version.

Benefits:

* Upgrade tracking
* Rollback identification
* Troubleshooting

---

## Label 6

```yaml
app.kubernetes.io/part-of: opentelemetry-demo
```

Indicates that CartService belongs to the overall OpenTelemetry Demo application.

---

# spec Section

```yaml
spec:
```

This section defines how the Service behaves.

---

# Service Type

```yaml
type: ClusterIP
```

### Meaning

Expose CartService only inside the Kubernetes cluster.

Accessible from:

```text
Frontend Pod
Checkout Pod
Product Catalog Pod
```

Not accessible from:

```text
Internet
Browser
Laptop
```

---

# Why ClusterIP?

CartService is an internal backend service.

Users never directly open CartService.

Users interact with:

```text
Browser
   │
   ▼
Frontend
   │
   ▼
CartService
```

Therefore ClusterIP is the safest and most appropriate choice.

---

# Ports Section

```yaml
ports:
```

Defines which ports the Service exposes.

---

## Service Port

```yaml
port: 8080
```

The Service listens on port 8080.

Clients connect using:

```text
opentelemetry-demo-cartservice:8080
```

---

## Port Name

```yaml
name: tcp-service
```

A logical name for the port.

Useful when:

* Multiple ports exist
* Service Mesh is used
* Monitoring tools inspect traffic

---

## Target Port

```yaml
targetPort: 8080
```

Traffic received on Service port 8080 is forwarded to:

```text
Container Port 8080
```

---

# Traffic Flow

```text
Frontend
    │
    ▼
Service Port 8080
    │
    ▼
Target Port 8080
    │
    ▼
CartService Container
```

---

# Selector Section

The selector is the heart of the Service.

```yaml
selector:
```

It determines:

```text
Which Pods receive traffic?
```

---

## Selector Value

```yaml
opentelemetry.io/name: opentelemetry-demo-cartservice
```

Kubernetes searches for Pods with exactly the same label.

Example Pod:

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-cartservice
```

Match found.

Traffic is routed to that Pod.

---
# Chapter 5: Service – opentelemetry-demo-checkoutservice

The next Kubernetes resource is:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: opentelemetry-demo-checkoutservice
  labels:
    opentelemetry.io/name: opentelemetry-demo-checkoutservice
    app.kubernetes.io/instance: opentelemetry-demo
    app.kubernetes.io/component: checkoutservice
    app.kubernetes.io/name: opentelemetry-demo-checkoutservice
    app.kubernetes.io/version: "1.12.0"
    app.kubernetes.io/part-of: opentelemetry-demo
spec:
  type: ClusterIP
  ports:
    - port: 8080
      name: tcp-service
      targetPort: 8080
  selector:
    opentelemetry.io/name: opentelemetry-demo-checkoutservice
```

---

# What is CheckoutService?

CheckoutService is one of the most important microservices in the application.

Its responsibility is to process the user's order when they click:

```text
Place Order
```

CheckoutService acts as an orchestrator.

It communicates with multiple backend services to complete an order.

---

# Checkout Flow in the Application

When a user clicks **Checkout**, the following process happens:

```text
User
 │
 ▼
Frontend
 │
 ▼
CheckoutService
 │
 ├── CartService
 │
 ├── ProductCatalogService
 │
 ├── PaymentService
 │
 ├── ShippingService
 │
 ├── EmailService
 │
 └── CurrencyService
```

---

# What Does CheckoutService Actually Do?

### Step 1: Retrieve Cart

CheckoutService asks CartService:

```text
Give me all products
currently in the cart.
```

---

### Step 2: Validate Products

CheckoutService contacts ProductCatalogService:

```text
Are these products valid?
```

---

### Step 3: Calculate Price

It may communicate with CurrencyService.

```text
Convert USD to INR
Convert USD to EUR
```

depending on customer preference.

---

### Step 4: Process Payment

CheckoutService calls PaymentService.

```text
Charge customer
```

---

### Step 5: Arrange Shipping

CheckoutService contacts ShippingService.

```text
Create shipment
```

---

### Step 6: Send Confirmation

CheckoutService contacts EmailService.

```text
Send order confirmation email
```

---

# Why Does CheckoutService Need a Kubernetes Service?

CheckoutService runs inside Pods.

Example:

```text
checkout-pod-abc
```

with IP:

```text
10.1.5.20
```

If the Pod crashes:

```text
checkout-pod-def
```

may be created with:

```text
10.1.5.40
```

The IP changes.

Other services cannot depend on changing IPs.

Therefore Kubernetes creates:

```text
opentelemetry-demo-checkoutservice
```

as a permanent endpoint.

---

# Communication Example

Instead of:

```text
10.1.5.20:8080
```

other services use:

```text
opentelemetry-demo-checkoutservice:8080
```

Kubernetes automatically routes traffic.

---

# YAML Breakdown

---

## Source Comment

```yaml
# Source: opentelemetry-demo/templates/component.yaml
```

This is a Helm-generated comment.

It tells us:

```text
This Service was generated
from component.yaml template.
```

Kubernetes ignores comments completely.

---

# apiVersion

```yaml
apiVersion: v1
```

## Purpose

Tells Kubernetes which API version manages this resource.

For Services:

```yaml
apiVersion: v1
```

is the standard version.

---

# kind

```yaml
kind: Service
```

## Purpose

Instructs Kubernetes:

```text
Create a Service object.
```

This resource is not:

* Deployment
* ConfigMap
* Secret
* Pod

It is specifically a Service.

---

# metadata

```yaml
metadata:
```

Contains identifying information about the Service.

---

## Service Name

```yaml
name: opentelemetry-demo-checkoutservice
```

This is the Service's unique name.

Kubernetes automatically creates DNS for it.

Applications can access it using:

```text
opentelemetry-demo-checkoutservice
```

instead of Pod IP addresses.

---

# Labels

Labels are metadata tags.

They help Kubernetes organize resources.

---

## OpenTelemetry Label

```yaml
opentelemetry.io/name: opentelemetry-demo-checkoutservice
```

Identifies the resource as the CheckoutService component.

---

## Instance Label

```yaml
app.kubernetes.io/instance: opentelemetry-demo
```

Identifies the application instance.

Useful if multiple environments exist.

Example:

```text
dev
test
production
```

---

## Component Label

```yaml
app.kubernetes.io/component: checkoutservice
```

One of the most important labels.

It identifies:

```text
Which microservice is this?
```

Answer:

```text
checkoutservice
```

---

## Application Name Label

```yaml
app.kubernetes.io/name: opentelemetry-demo-checkoutservice
```

Provides a standard application name.

Used by:

* Helm
* Monitoring tools
* Dashboards

---

## Version Label

```yaml
app.kubernetes.io/version: "1.12.0"
```

Specifies application version.

Useful during:

* Upgrades
* Rollbacks
* Troubleshooting

---

## Part-of Label

```yaml
app.kubernetes.io/part-of: opentelemetry-demo
```

Shows that CheckoutService belongs to the larger OpenTelemetry Demo application.

---

# spec Section

```yaml
spec:
```

Defines how the Service behaves.

---

# Service Type

```yaml
type: ClusterIP
```

This means:

```text
Internal Kubernetes Service
```

Only Pods inside the cluster can access it.

---

# Why ClusterIP?

CheckoutService is a backend service.

Users should not directly access it.

Correct flow:

```text
Browser
   │
   ▼
Frontend
   │
   ▼
CheckoutService
```

Not:

```text
Browser
   │
   ▼
CheckoutService
```

Therefore ClusterIP is the correct choice.

---

# Ports Section

```yaml
ports:
```

Defines how traffic enters the Service.

---

## Service Port

```yaml
port: 8080
```

Clients connect using:

```text
opentelemetry-demo-checkoutservice:8080
```

---

## Port Name

```yaml
name: tcp-service
```

A friendly name for the port.

Useful when:

* Multiple ports exist
* Service Mesh is used
* Monitoring systems inspect traffic

---

## Target Port

```yaml
targetPort: 8080
```

The Service forwards requests to container port 8080.

---

# Traffic Flow

```text
Client
  │
  ▼
Service Port 8080
  │
  ▼
Target Port 8080
  │
  ▼
CheckoutService Container
```

---

# Selector Section

This is the most critical part of the Service.

```yaml
selector:
```

Selectors determine:

```text
Which Pods should receive traffic?
```

---

## Selector Label

```yaml
opentelemetry.io/name: opentelemetry-demo-checkoutservice
```

Kubernetes searches for Pods containing this label.

Example:

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-checkoutservice
```

If a Pod matches:

```text
Service
   │
   ▼
Checkout Pod
```

Traffic is routed to it.

---

# Chapter 6: Service – opentelemetry-demo-currencyservice

The next Kubernetes Service is:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: opentelemetry-demo-currencyservice
spec:
  type: ClusterIP
  ports:
    - port: 8080
      targetPort: 8080
  selector:
    opentelemetry.io/name: opentelemetry-demo-currencyservice
```

---

# What is CurrencyService?

CurrencyService is responsible for currency conversion within the application.

In an e-commerce platform, customers may belong to different countries and want to see prices in their local currency.

For example:

```text
Product Price = $100
```

Customer from India wants:

```text
₹8,500
```

Customer from Europe wants:

```text
€90
```

CurrencyService performs these conversions.

---

# Where is CurrencyService Used?

The primary consumer is CheckoutService.

Example:

```text
User
 │
 ▼
Frontend
 │
 ▼
CheckoutService
 │
 ▼
CurrencyService
```

CheckoutService sends:

```text
Convert USD to INR
```

CurrencyService returns:

```text
8500 INR
```

---

# Why Does CurrencyService Need a Service?

CurrencyService Pods may restart.

Example:

```text
currency-pod-1
IP = 10.1.2.10
```

After restart:

```text
currency-pod-2
IP = 10.1.2.25
```

IP changes.

Instead of calling Pod IPs, applications call:

```text
opentelemetry-demo-currencyservice
```

which remains constant.

---

# Metadata Section

## Name

```yaml
name: opentelemetry-demo-currencyservice
```

Creates an internal DNS name:

```text
opentelemetry-demo-currencyservice
```

Applications access it as:

```text
opentelemetry-demo-currencyservice:8080
```

---

## Important Label

```yaml
app.kubernetes.io/component: currencyservice
```

This identifies the business function of this microservice.

Meaning:

```text
This service handles
currency conversion operations.
```

---

# Service Type

```yaml
type: ClusterIP
```

Meaning:

```text
Accessible only within Kubernetes cluster
```

Not exposed to:

* Internet
* Browsers
* External users

Accessible to:

* CheckoutService
* Frontend
* Other internal services

---

# Port Configuration

```yaml
port: 8080
targetPort: 8080
```

Traffic Flow:

```text
CheckoutService
      │
      ▼
CurrencyService:8080
      │
      ▼
CurrencyService Pod:8080
```

---

# Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-currencyservice
```

Purpose:

```text
Find CurrencyService Pods
and send traffic to them.
```

Only Pods with matching labels receive requests.

---

# Chapter 7: Service – opentelemetry-demo-emailservice

The next Service is:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: opentelemetry-demo-emailservice
spec:
  type: ClusterIP
  ports:
    - port: 8080
      targetPort: 8080
  selector:
    opentelemetry.io/name: opentelemetry-demo-emailservice
```

---

# What is EmailService?

EmailService is responsible for sending emails to customers.

Typical emails include:

* Order confirmation
* Shipping notification
* Purchase receipt
* Transaction updates

---

# Where is EmailService Used?

After a successful checkout:

```text
User Places Order
        │
        ▼
CheckoutService
        │
        ▼
EmailService
        │
        ▼
Confirmation Email Sent
```

---

# Example Checkout Flow

```text
User
 │
 ▼
CheckoutService
 │
 ├── PaymentService
 ├── ShippingService
 └── EmailService
        │
        ▼
Send Order Confirmation
```

---

# Why Does EmailService Need a Service?

Pods are temporary.

Example:

```text
email-pod-1
IP = 10.1.3.10
```

After restart:

```text
email-pod-2
IP = 10.1.3.55
```

The IP changes.

Applications instead use:

```text
opentelemetry-demo-emailservice
```

which never changes.

---

# Metadata Section

## Name

```yaml
name: opentelemetry-demo-emailservice
```

Creates DNS entry:

```text
opentelemetry-demo-emailservice
```

Other services communicate using:

```text
opentelemetry-demo-emailservice:8080
```

---

## Important Label

```yaml
app.kubernetes.io/component: emailservice
```

Identifies this microservice as:

```text
Email Processing Service
```

Useful for:

* Monitoring
* Logging
* Troubleshooting
* Metrics

---

# Service Type

```yaml
type: ClusterIP
```

Meaning:

```text
Internal-only Service
```

EmailService is not directly exposed to users.

Only internal services communicate with it.

---

# Port Configuration

```yaml
port: 8080
targetPort: 8080
```

Traffic Flow:

```text
CheckoutService
      │
      ▼
EmailService:8080
      │
      ▼
EmailService Pod:8080
```

---

# Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-emailservice
```

Kubernetes finds Pods with:

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-emailservice
```

and routes traffic to them.

---

# Chapter 8: Service – opentelemetry-demo-flagd

The next Service is different from the previous ones because it exposes **two ports instead of one**.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: opentelemetry-demo-flagd
spec:
  type: ClusterIP
  ports:
    - port: 8013
      name: tcp-service
      targetPort: 8013

    - port: 4000
      name: tcp-service-0
      targetPort: 4000

  selector:
    opentelemetry.io/name: opentelemetry-demo-flagd
```

---

# What is Flagd?

Flagd is a **Feature Flag Management Service**.

Feature Flags allow developers to:

* Enable new features
* Disable features
* Test functionality
* Run A/B testing

without redeploying the application.

---

## Real-World Example

Suppose developers create a new payment option.

Instead of deploying separate versions:

```text
Version 1
Version 2
```

they use a feature flag.

```text
NewPaymentFeature = OFF
```

Users see old behavior.

Later:

```text
NewPaymentFeature = ON
```

Users see the new feature instantly.

No redeployment required.

---

# Where is Flagd Used?

In the OpenTelemetry Demo application:

```text
Frontend
    │
    ▼
Feature Flag Check
    │
    ▼
Flagd
```

Before displaying certain features, services ask Flagd:

```text
Is feature enabled?
```

Flagd returns:

```text
true
or
false
```

---

# Why Does Flagd Need a Service?

Like all applications, Flagd runs in Pods.

```text
flagd-pod
```

Pod IPs can change.

Therefore applications communicate using:

```text
opentelemetry-demo-flagd
```

instead of Pod IP addresses.

---

# Metadata Analysis

## Name

```yaml
name: opentelemetry-demo-flagd
```

Creates internal DNS:

```text
opentelemetry-demo-flagd
```

Applications can access:

```text
opentelemetry-demo-flagd:8013
```

or

```text
opentelemetry-demo-flagd:4000
```

depending on the protocol being used.

---

# Important Label

```yaml
app.kubernetes.io/component: flagd
```

Identifies this component as:

```text
Feature Flag Service
```

Useful in:

* Monitoring
* Logging
* Service discovery
* Dashboards

---

# Service Type

```yaml
type: ClusterIP
```

Meaning:

```text
Internal Kubernetes Service
```

Accessible only inside the cluster.

---

# Understanding Multiple Ports

This is the first service where we see:

```yaml
ports:
```

containing multiple entries.

---

## Port 1

```yaml
- port: 8013
  name: tcp-service
  targetPort: 8013
```

Traffic Flow:

```text
Application
     │
     ▼
Flagd Service:8013
     │
     ▼
Flagd Pod:8013
```

This port is typically used by applications communicating with Flagd.

---

## Port 2

```yaml
- port: 4000
  name: tcp-service-0
  targetPort: 4000
```

Traffic Flow:

```text
Application
     │
     ▼
Flagd Service:4000
     │
     ▼
Flagd Pod:4000
```

Flagd exposes another interface on this port.

---

# Why Multiple Ports?

Some applications expose different services on different ports.

Example:

```text
Port 8080 -> Application API

Port 9090 -> Metrics

Port 8443 -> Secure API
```

Similarly Flagd exposes two separate interfaces.

---

# Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-flagd
```

Kubernetes searches for Pods having:

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-flagd
```

and forwards traffic to them.

---

# Traffic Flow

```text
Frontend
    │
    ▼
Flagd Service
    │
    ▼
Flagd Pod
```

The Service hides Pod IP changes.

---

# Chapter 9: Service – opentelemetry-demo-frontend

The next Service is:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: opentelemetry-demo-frontend
spec:
  type: ClusterIP
  ports:
    - port: 8080
      name: tcp-service
      targetPort: 8080
  selector:
    opentelemetry.io/name: opentelemetry-demo-frontend
```

---

# What is Frontend?

Frontend is the user-facing application.

This is the web application customers interact with.

Users perform actions such as:

* Browse products
* View recommendations
* Add products to cart
* Checkout orders

through the Frontend.

---

# Application Architecture

```text
User
 │
 ▼
Frontend
 │
 ├── ProductCatalogService
 ├── CartService
 ├── CheckoutService
 ├── CurrencyService
 ├── RecommendationService
 └── AdService
```

Frontend acts as the entry point into the entire application.

---

# Why Does Frontend Need a Service?

Frontend Pods can restart.

Example:

```text
frontend-pod-1
IP = 10.1.5.10
```

After restart:

```text
frontend-pod-2
IP = 10.1.5.25
```

The IP changes.

Users and other services need a stable endpoint.

Therefore Kubernetes creates:

```text
opentelemetry-demo-frontend
```

---

# Metadata

## Name

```yaml
name: opentelemetry-demo-frontend
```

Creates DNS entry:

```text
opentelemetry-demo-frontend
```

This becomes the stable address of the Frontend service.

---

## Important Label

```yaml
app.kubernetes.io/component: frontend
```

Indicates:

```text
This resource belongs to
the Frontend component.
```

Useful for monitoring and troubleshooting.

---

# Service Type

```yaml
type: ClusterIP
```

Meaning:

```text
Internal Cluster Service
```

At first glance this may seem surprising because users access the frontend.

However, in many Kubernetes environments:

```text
Internet
    │
    ▼
Ingress
    │
    ▼
Frontend Service
    │
    ▼
Frontend Pods
```

The Frontend Service remains ClusterIP while an Ingress or LoadBalancer handles external traffic.

---

# Port Configuration

## Service Port

```yaml
port: 8080
```

Frontend listens on:

```text
opentelemetry-demo-frontend:8080
```

---

## Target Port

```yaml
targetPort: 8080
```

Requests are forwarded to:

```text
Frontend Container Port 8080
```

---

# Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-frontend
```

This tells Kubernetes:

```text
Find all Frontend Pods
and send traffic to them.
```

---

# Traffic Flow

```text
User Browser
      │
      ▼
Ingress / LoadBalancer
      │
      ▼
Frontend Service
      │
      ▼
Frontend Pod
```

---

# Why Frontend Is Important

Among all services discussed so far:

```text
AdService
CartService
CheckoutService
CurrencyService
EmailService
Flagd
Frontend
```

Frontend is the only component directly responsible for presenting the application UI to users.

All other services operate behind the scenes.

---

# Chapter 10: Service – opentelemetry-demo-frontendproxy

The next Kubernetes Service is:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: opentelemetry-demo-frontendproxy
  labels:
    opentelemetry.io/name: opentelemetry-demo-frontendproxy
    app.kubernetes.io/instance: opentelemetry-demo
    app.kubernetes.io/component: frontendproxy
    app.kubernetes.io/name: opentelemetry-demo-frontendproxy
    app.kubernetes.io/version: "1.12.0"
    app.kubernetes.io/part-of: opentelemetry-demo
spec:
  type: ClusterIP
  ports:
    - port: 8080
      name: tcp-service
      targetPort: 8080
  selector:
    opentelemetry.io/name: opentelemetry-demo-frontendproxy
```

---

# What is FrontendProxy?

FrontendProxy acts as a reverse proxy positioned in front of the Frontend application.

Think of it as a traffic controller.

```text
User Request
      │
      ▼
FrontendProxy
      │
      ▼
Frontend
```

Instead of users communicating directly with Frontend Pods, requests first reach FrontendProxy.

---

# Why Do We Need FrontendProxy?

A proxy can:

* Route requests
* Forward traffic
* Add security headers
* Perform load balancing
* Collect telemetry data
* Control access

In the OpenTelemetry Demo, FrontendProxy helps simulate real-world production architectures.

---

# Application Flow

```text
Browser
   │
   ▼
FrontendProxy
   │
   ▼
Frontend
   │
   ▼
Backend Services
```

Without FrontendProxy:

```text
Browser
   │
   ▼
Frontend
```

With FrontendProxy:

```text
Browser
   │
   ▼
FrontendProxy
   │
   ▼
Frontend
```

This additional layer provides more control over traffic.

---

# Metadata Section

## Name

```yaml
name: opentelemetry-demo-frontendproxy
```

Creates a DNS entry:

```text
opentelemetry-demo-frontendproxy
```

Applications inside Kubernetes can access:

```text
http://opentelemetry-demo-frontendproxy:8080
```

---

# Labels Section

Labels help Kubernetes identify, organize, and manage resources.

---

## Label 1

```yaml
opentelemetry.io/name: opentelemetry-demo-frontendproxy
```

Identifies the resource as the FrontendProxy component.

Useful for:

```bash
kubectl get all -l opentelemetry.io/name=opentelemetry-demo-frontendproxy
```

---

## Label 2

```yaml
app.kubernetes.io/instance: opentelemetry-demo
```

Identifies the application instance.

Useful when running:

```text
opentelemetry-demo-dev
opentelemetry-demo-test
opentelemetry-demo-prod
```

---

## Label 3

```yaml
app.kubernetes.io/component: frontendproxy
```

Very important.

This tells us:

```text
Component Name = frontendproxy
```

Used heavily by:

* Monitoring systems
* Dashboards
* Logging tools

---

## Label 4

```yaml
app.kubernetes.io/name: opentelemetry-demo-frontendproxy
```

Application name label.

Provides a standard Kubernetes naming convention.

---

## Label 5

```yaml
app.kubernetes.io/version: "1.12.0"
```

Application version.

Useful during:

* Upgrades
* Rollbacks
* Troubleshooting

---

## Label 6

```yaml
app.kubernetes.io/part-of: opentelemetry-demo
```

Indicates this service belongs to the OpenTelemetry Demo application.

---

# Service Type

```yaml
type: ClusterIP
```

Meaning:

```text
Internal Kubernetes Service
```

Only accessible inside the cluster.

---

# Port Configuration

```yaml
ports:
  - port: 8080
    name: tcp-service
    targetPort: 8080
```

Traffic Flow:

```text
Client
  │
  ▼
Service Port 8080
  │
  ▼
FrontendProxy Container Port 8080
```

---

# Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-frontendproxy
```

Kubernetes searches for Pods with:

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-frontendproxy
```

Traffic is sent only to matching Pods.

---

# Chapter 11: Service – opentelemetry-demo-imageprovider

The next Service is:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: opentelemetry-demo-imageprovider
  labels:
    opentelemetry.io/name: opentelemetry-demo-imageprovider
    app.kubernetes.io/instance: opentelemetry-demo
    app.kubernetes.io/component: imageprovider
    app.kubernetes.io/name: opentelemetry-demo-imageprovider
    app.kubernetes.io/version: "1.12.0"
    app.kubernetes.io/part-of: opentelemetry-demo
spec:
  type: ClusterIP
  ports:
    - port: 8081
      name: tcp-service
      targetPort: 8081
  selector:
    opentelemetry.io/name: opentelemetry-demo-imageprovider
```

---

# What is ImageProvider?

ImageProvider is responsible for supplying product images used throughout the online store.

Examples:

```text
Laptop Image
Phone Image
Watch Image
Headphone Image
```

Instead of storing images directly inside Frontend, a dedicated service provides them.

---

# Why Separate ImageProvider?

In microservice architecture, responsibilities are separated.

```text
Frontend
   │
   ├── Product Information
   ├── Cart Data
   └── Product Images
```

Product images can be managed independently.

Benefits:

* Better scalability
* Easier maintenance
* Independent deployment
* Faster image handling

---

# Application Flow

```text
User Opens Product Page
          │
          ▼
Frontend
          │
          ▼
ImageProvider
          │
          ▼
Returns Product Image
```

---

# Metadata Section

## Name

```yaml
name: opentelemetry-demo-imageprovider
```

Creates DNS:

```text
opentelemetry-demo-imageprovider
```

Applications access:

```text
http://opentelemetry-demo-imageprovider:8081
```

---

# Labels Section

## Label 1

```yaml
opentelemetry.io/name: opentelemetry-demo-imageprovider
```

Identifies the ImageProvider component.

---

## Label 2

```yaml
app.kubernetes.io/instance: opentelemetry-demo
```

Identifies the application instance.

---

## Label 3

```yaml
app.kubernetes.io/component: imageprovider
```

Very important.

Defines:

```text
Component = imageprovider
```

Used for monitoring and troubleshooting.

---

## Label 4

```yaml
app.kubernetes.io/name: opentelemetry-demo-imageprovider
```

Application name label.

---

## Label 5

```yaml
app.kubernetes.io/version: "1.12.0"
```

Application version.

---

## Label 6

```yaml
app.kubernetes.io/part-of: opentelemetry-demo
```

Indicates this resource belongs to the OpenTelemetry Demo application.

---

# Service Type

```yaml
type: ClusterIP
```

Meaning:

```text
Internal Service Only
```

Accessible only within Kubernetes.

---

# Port Configuration

Unlike previous services using port 8080:

```yaml
port: 8081
targetPort: 8081
```

This means:

```text
ImageProvider Service → Port 8081
ImageProvider Container → Port 8081
```

Traffic Flow:

```text
Frontend
    │
    ▼
ImageProvider Service:8081
    │
    ▼
ImageProvider Pod:8081
```

---

# Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-imageprovider
```

Kubernetes finds Pods having:

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-imageprovider
```

and routes traffic to them.

---

You're right. I should not have shortened the labels section. Since you're creating **project documentation**, every label should be documented consistently for every service, even if the explanation is repetitive. In real documentation, readers may jump directly to a specific service section.

Let's do the next two services properly.

# Chapter 12: Service – opentelemetry-demo-kafka

```yaml
apiVersion: v1
kind: Service
metadata:
  name: opentelemetry-demo-kafka
  labels:
    opentelemetry.io/name: opentelemetry-demo-kafka
    app.kubernetes.io/instance: opentelemetry-demo
    app.kubernetes.io/component: kafka
    app.kubernetes.io/name: opentelemetry-demo-kafka
    app.kubernetes.io/version: "1.12.0"
    app.kubernetes.io/part-of: opentelemetry-demo
spec:
  type: ClusterIP
  ports:
    - port: 9092
      name: plaintext
      targetPort: 9092
    - port: 9093
      name: controller
      targetPort: 9093
  selector:
    opentelemetry.io/name: opentelemetry-demo-kafka
```

---

# What is Kafka?

Kafka is a distributed event streaming platform.

Instead of services communicating directly:

```text
Service A
   │
   ▼
Service B
```

they can communicate through Kafka.

```text
Service A
   │
   ▼
Kafka
   │
   ▼
Service B
```

This creates loose coupling between services.

---

# Why is Kafka Used?

Kafka handles:

* Event Streaming
* Message Queues
* Asynchronous Communication
* Data Pipelines
* Real-time Event Processing

In OpenTelemetry Demo, Kafka is commonly used for telemetry and event-driven communication.

---

# Metadata Section

## Name

```yaml
name: opentelemetry-demo-kafka
```

Creates internal DNS:

```text
opentelemetry-demo-kafka
```

Applications connect using:

```text
opentelemetry-demo-kafka:9092
```

---

# Labels Section

## Label 1

```yaml
opentelemetry.io/name: opentelemetry-demo-kafka
```

Identifies the resource as the Kafka component.

Used for resource discovery.

---

## Label 2

```yaml
app.kubernetes.io/instance: opentelemetry-demo
```

Indicates this Kafka instance belongs to the OpenTelemetry Demo deployment.

---

## Label 3

```yaml
app.kubernetes.io/component: kafka
```

Very important.

Defines:

```text
Component = kafka
```

Useful for:

* Monitoring
* Logging
* Dashboards
* Resource filtering

---

## Label 4

```yaml
app.kubernetes.io/name: opentelemetry-demo-kafka
```

Application name label.

Provides standardized naming.

---

## Label 5

```yaml
app.kubernetes.io/version: "1.12.0"
```

Application version.

Helps during:

* Upgrades
* Rollbacks
* Troubleshooting

---

## Label 6

```yaml
app.kubernetes.io/part-of: opentelemetry-demo
```

Indicates Kafka belongs to the OpenTelemetry Demo application.

---

# Service Type

```yaml
type: ClusterIP
```

Kafka is accessible only within the Kubernetes cluster.

---

# Ports Section

This Service exposes **two ports**.

## Port 9092

```yaml
- port: 9092
  name: plaintext
  targetPort: 9092
```

This is the main Kafka client communication port.

Applications connect to Kafka using:

```text
Producer
   │
   ▼
Kafka : 9092
```

or

```text
Consumer
   │
   ▼
Kafka : 9092
```

---

## Port 9093

```yaml
- port: 9093
  name: controller
  targetPort: 9093
```

Used internally by Kafka controllers.

Purpose:

```text
Cluster Coordination
Leader Election
Broker Management
```

Normally application developers do not connect to this port directly.

---

# Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-kafka
```

The Service routes traffic only to Pods having:

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-kafka
```

---

# Traffic Flow

```text
Application
     │
     ▼
Kafka Service
     │
     ▼
Kafka Pod
```

---

# Chapter 13: Service – opentelemetry-demo-loadgenerator

```yaml
apiVersion: v1
kind: Service
metadata:
  name: opentelemetry-demo-loadgenerator
  labels:
    opentelemetry.io/name: opentelemetry-demo-loadgenerator
    app.kubernetes.io/instance: opentelemetry-demo
    app.kubernetes.io/component: loadgenerator
    app.kubernetes.io/name: opentelemetry-demo-loadgenerator
    app.kubernetes.io/version: "1.12.0"
    app.kubernetes.io/part-of: opentelemetry-demo
spec:
  type: ClusterIP
  ports:
    - port: 8089
      name: tcp-service
      targetPort: 8089
  selector:
    opentelemetry.io/name: opentelemetry-demo-loadgenerator
```

---

# What is LoadGenerator?

LoadGenerator is a special service used only for testing and demonstrations.

It automatically generates traffic against the application.

Instead of requiring real users:

```text
User
 │
 ▼
Frontend
```

LoadGenerator simulates users.

```text
LoadGenerator
     │
     ▼
Frontend
```

---

# Why Do We Need LoadGenerator?

OpenTelemetry is an observability demo.

Without traffic:

```text
No Requests
No Metrics
No Traces
No Logs
```

Observability dashboards would appear empty.

LoadGenerator continuously generates requests so telemetry data is always available.

---

# Simulated User Actions

LoadGenerator may perform:

```text
Browse Products
View Product Details
Add To Cart
Checkout Orders
Search Products
```

automatically.

---

# Metadata Section

## Name

```yaml
name: opentelemetry-demo-loadgenerator
```

Creates DNS:

```text
opentelemetry-demo-loadgenerator
```

---

# Labels Section

## Label 1

```yaml
opentelemetry.io/name: opentelemetry-demo-loadgenerator
```

Identifies the LoadGenerator resource.

---

## Label 2

```yaml
app.kubernetes.io/instance: opentelemetry-demo
```

Associates the resource with the OpenTelemetry Demo deployment.

---

## Label 3

```yaml
app.kubernetes.io/component: loadgenerator
```

Defines:

```text
Component = loadgenerator
```

Useful for filtering and monitoring.

---

## Label 4

```yaml
app.kubernetes.io/name: opentelemetry-demo-loadgenerator
```

Application name label.

---

## Label 5

```yaml
app.kubernetes.io/version: "1.12.0"
```

Specifies deployed version.

---

## Label 6

```yaml
app.kubernetes.io/part-of: opentelemetry-demo
```

Indicates LoadGenerator belongs to the OpenTelemetry Demo application.

---

# Service Type

```yaml
type: ClusterIP
```

Accessible only inside the cluster.

---

# Port Configuration

```yaml
- port: 8089
  name: tcp-service
  targetPort: 8089
```

Traffic Flow:

```text
Client
   │
   ▼
LoadGenerator Service : 8089
   │
   ▼
LoadGenerator Pod : 8089
```

---

# Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-loadgenerator
```

Matches Pods with:

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-loadgenerator
```

---

# Chapter 14: Service – opentelemetry-demo-paymentservice

The next Kubernetes Service is:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: opentelemetry-demo-paymentservice
  labels:
    opentelemetry.io/name: opentelemetry-demo-paymentservice
    app.kubernetes.io/instance: opentelemetry-demo
    app.kubernetes.io/component: paymentservice
    app.kubernetes.io/name: opentelemetry-demo-paymentservice
    app.kubernetes.io/version: "1.12.0"
    app.kubernetes.io/part-of: opentelemetry-demo
spec:
  type: ClusterIP
  ports:
    - port: 8080
      name: tcp-service
      targetPort: 8080
  selector:
    opentelemetry.io/name: opentelemetry-demo-paymentservice
```

---

# What is PaymentService?

PaymentService is responsible for processing customer payments during checkout.

When a customer clicks:

```text
Place Order
```

CheckoutService communicates with PaymentService to complete the payment transaction.

---

# Payment Flow

```text
User
 │
 ▼
Frontend
 │
 ▼
CheckoutService
 │
 ▼
PaymentService
 │
 ▼
Payment Approved / Rejected
```

---

# Responsibilities of PaymentService

PaymentService handles:

* Payment authorization
* Payment validation
* Transaction processing
* Payment status response

Example:

```text
Order Amount = $100

PaymentService
      │
      ▼
Validate Card
      │
      ▼
Approve Payment
```

---

# Metadata Section

## Name

```yaml
name: opentelemetry-demo-paymentservice
```

Creates a Kubernetes DNS name:

```text
opentelemetry-demo-paymentservice
```

Other services connect using:

```text
opentelemetry-demo-paymentservice:8080
```

---

# Labels Section

## Label 1

```yaml
opentelemetry.io/name: opentelemetry-demo-paymentservice
```

Identifies this resource as the PaymentService component.

Used for filtering:

```bash
kubectl get all -l opentelemetry.io/name=opentelemetry-demo-paymentservice
```

---

## Label 2

```yaml
app.kubernetes.io/instance: opentelemetry-demo
```

Specifies the application instance.

Meaning:

```text
This PaymentService belongs to the
OpenTelemetry Demo deployment.
```

---

## Label 3

```yaml
app.kubernetes.io/component: paymentservice
```

One of the most important labels.

Defines:

```text
Component = paymentservice
```

Used by:

* Monitoring tools
* Dashboards
* Logging systems

---

## Label 4

```yaml
app.kubernetes.io/name: opentelemetry-demo-paymentservice
```

Standard Kubernetes application name label.

---

## Label 5

```yaml
app.kubernetes.io/version: "1.12.0"
```

Application version.

Useful during:

* Upgrades
* Rollbacks
* Troubleshooting

---

## Label 6

```yaml
app.kubernetes.io/part-of: opentelemetry-demo
```

Indicates this service belongs to the overall OpenTelemetry Demo application.

---

# Service Type

```yaml
type: ClusterIP
```

Meaning:

```text
Internal Kubernetes Service
```

Only accessible within the cluster.

---

# Port Configuration

```yaml
ports:
  - port: 8080
    name: tcp-service
    targetPort: 8080
```

Traffic Flow:

```text
CheckoutService
       │
       ▼
PaymentService:8080
       │
       ▼
PaymentService Pod:8080
```

---

# Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-paymentservice
```

Kubernetes routes traffic only to Pods containing:

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-paymentservice
```

---

# Why PaymentService is Important?

Without PaymentService:

```text
Cart Available
Products Available
Checkout Available

But No Payment Processing
```

The customer cannot complete a purchase.

PaymentService is therefore a critical business service.

---

# Chapter 15: Service – opentelemetry-demo-productcatalogservice

The next Kubernetes Service is:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: opentelemetry-demo-productcatalogservice
  labels:
    opentelemetry.io/name: opentelemetry-demo-productcatalogservice
    app.kubernetes.io/instance: opentelemetry-demo
    app.kubernetes.io/component: productcatalogservice
    app.kubernetes.io/name: opentelemetry-demo-productcatalogservice
    app.kubernetes.io/version: "1.12.0"
    app.kubernetes.io/part-of: opentelemetry-demo
spec:
  type: ClusterIP
  ports:
    - port: 8080
      name: tcp-service
      targetPort: 8080
  selector:
    opentelemetry.io/name: opentelemetry-demo-productcatalogservice
```

---

# What is ProductCatalogService?

ProductCatalogService stores and provides product information.

Whenever a customer browses the store, the Frontend obtains product data from ProductCatalogService.

---

# Example Product Information

```text
Product Name
Description
Price
Category
Availability
Product ID
```

Example:

```text
Product:
  Laptop

Price:
  $1200

Category:
  Electronics
```

This information originates from ProductCatalogService.

---

# Application Flow

```text
User
 │
 ▼
Frontend
 │
 ▼
ProductCatalogService
 │
 ▼
Product Data Returned
```

---

# Why is ProductCatalogService Important?

Almost every user action depends on product information.

Without ProductCatalogService:

```text
No Products Displayed
No Product Search
No Product Details
```

The online store becomes unusable.

---

# Metadata Section

## Name

```yaml
name: opentelemetry-demo-productcatalogservice
```

Creates DNS:

```text
opentelemetry-demo-productcatalogservice
```

Applications connect using:

```text
opentelemetry-demo-productcatalogservice:8080
```

---

# Labels Section

## Label 1

```yaml
opentelemetry.io/name: opentelemetry-demo-productcatalogservice
```

Identifies the ProductCatalogService component.

---

## Label 2

```yaml
app.kubernetes.io/instance: opentelemetry-demo
```

Specifies the deployment instance.

Meaning:

```text
This resource belongs to the
OpenTelemetry Demo application.
```

---

## Label 3

```yaml
app.kubernetes.io/component: productcatalogservice
```

Very important label.

Defines:

```text
Component = productcatalogservice
```

Useful for:

* Monitoring
* Logging
* Dashboards
* Troubleshooting

---

## Label 4

```yaml
app.kubernetes.io/name: opentelemetry-demo-productcatalogservice
```

Standard Kubernetes application naming label.

---

## Label 5

```yaml
app.kubernetes.io/version: "1.12.0"
```

Specifies deployed version.

Useful during upgrades and rollback operations.

---

## Label 6

```yaml
app.kubernetes.io/part-of: opentelemetry-demo
```

Shows ProductCatalogService belongs to the OpenTelemetry Demo application.

---

# Service Type

```yaml
type: ClusterIP
```

Meaning:

```text
Accessible only from inside the cluster.
```

External users never directly access ProductCatalogService.

Instead:

```text
Browser
   │
   ▼
Frontend
   │
   ▼
ProductCatalogService
```

---

# Port Configuration

```yaml
ports:
  - port: 8080
    name: tcp-service
    targetPort: 8080
```

Traffic Flow:

```text
Frontend
    │
    ▼
ProductCatalogService:8080
    │
    ▼
ProductCatalog Pod:8080
```

---

# Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-productcatalogservice
```

Matches Pods containing:

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-productcatalogservice
```

The Service routes traffic to those Pods.

---

# Chapter 16: Service – opentelemetry-demo-quoteservice

The next Kubernetes Service is:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: opentelemetry-demo-quoteservice
  labels:
    opentelemetry.io/name: opentelemetry-demo-quoteservice
    app.kubernetes.io/instance: opentelemetry-demo
    app.kubernetes.io/component: quoteservice
    app.kubernetes.io/name: opentelemetry-demo-quoteservice
    app.kubernetes.io/version: "1.12.0"
    app.kubernetes.io/part-of: opentelemetry-demo
spec:
  type: ClusterIP
  ports:
    - port: 8080
      name: tcp-service
      targetPort: 8080
  selector:
    opentelemetry.io/name: opentelemetry-demo-quoteservice
```

---

# What is QuoteService?

QuoteService provides random quotes that appear within the OpenTelemetry Demo application.

Its purpose is mainly educational and demonstrational rather than core e-commerce functionality.

Example response:

```text
"Success is not final,
failure is not fatal."
```

or

```text
"Learning never exhausts the mind."
```

The Frontend requests quotes from this service and displays them to users.

---

# Why Does QuoteService Exist?

The OpenTelemetry Demo is designed to demonstrate:

* Distributed tracing
* Metrics collection
* Logging
* Service-to-service communication

QuoteService adds another independent microservice that generates telemetry data.

---

# Application Flow

```text
User
 │
 ▼
Frontend
 │
 ▼
QuoteService
 │
 ▼
Random Quote
```

---

# Metadata Section

## Name

```yaml
name: opentelemetry-demo-quoteservice
```

Creates internal Kubernetes DNS:

```text
opentelemetry-demo-quoteservice
```

Applications connect using:

```text
opentelemetry-demo-quoteservice:8080
```

---

# Labels Section

## Label 1

```yaml
opentelemetry.io/name: opentelemetry-demo-quoteservice
```

Identifies the QuoteService component.

Useful for:

```bash
kubectl get all -l opentelemetry.io/name=opentelemetry-demo-quoteservice
```

---

## Label 2

```yaml
app.kubernetes.io/instance: opentelemetry-demo
```

Identifies the application instance.

Meaning:

```text
This QuoteService belongs to
the OpenTelemetry Demo deployment.
```

---

## Label 3

```yaml
app.kubernetes.io/component: quoteservice
```

Very important label.

Defines:

```text
Component = quoteservice
```

Used by:

* Monitoring systems
* Logging systems
* Dashboards
* Resource filtering

---

## Label 4

```yaml
app.kubernetes.io/name: opentelemetry-demo-quoteservice
```

Standard Kubernetes application naming label.

---

## Label 5

```yaml
app.kubernetes.io/version: "1.12.0"
```

Specifies deployed version.

Useful during:

* Upgrades
* Rollbacks
* Troubleshooting

---

## Label 6

```yaml
app.kubernetes.io/part-of: opentelemetry-demo
```

Shows that this service is part of the OpenTelemetry Demo application.

---

# Service Type

```yaml
type: ClusterIP
```

Meaning:

```text
Internal Kubernetes Service
```

Only Pods inside the cluster can access it.

---

# Port Configuration

```yaml
ports:
  - port: 8080
    name: tcp-service
    targetPort: 8080
```

Traffic Flow:

```text
Frontend
   │
   ▼
QuoteService:8080
   │
   ▼
QuoteService Pod:8080
```

---

# Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-quoteservice
```

Kubernetes finds Pods with:

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-quoteservice
```

and routes requests to them.

---

# Chapter 17: Service – opentelemetry-demo-recommendationservice

The next Kubernetes Service is:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: opentelemetry-demo-recommendationservice
  labels:
    opentelemetry.io/name: opentelemetry-demo-recommendationservice
    app.kubernetes.io/instance: opentelemetry-demo
    app.kubernetes.io/component: recommendationservice
    app.kubernetes.io/name: opentelemetry-demo-recommendationservice
    app.kubernetes.io/version: "1.12.0"
    app.kubernetes.io/part-of: opentelemetry-demo
spec:
  type: ClusterIP
  ports:
    - port: 8080
      name: tcp-service
      targetPort: 8080
  selector:
    opentelemetry.io/name: opentelemetry-demo-recommendationservice
```

---

# What is RecommendationService?

RecommendationService suggests products that a customer may be interested in purchasing.

When a customer views a product, the service generates recommendations such as:

```text
Customers who viewed this item
also viewed:
```

Example:

```text
Laptop
   │
   ▼
Recommended:
- Mouse
- Keyboard
- Laptop Bag
```

---

# Why Is RecommendationService Important?

Recommendation systems increase:

* Customer engagement
* Product discovery
* Average order value
* Cross-selling opportunities

Many large e-commerce platforms use recommendation engines extensively.

---

# Application Flow

```text
User Views Product
         │
         ▼
Frontend
         │
         ▼
RecommendationService
         │
         ▼
Suggested Products
```

---

# Example Scenario

Customer opens:

```text
Gaming Laptop
```

RecommendationService may return:

```text
Gaming Mouse
Gaming Headset
Mechanical Keyboard
Laptop Cooling Pad
```

The Frontend displays these recommendations.

---

# Metadata Section

## Name

```yaml
name: opentelemetry-demo-recommendationservice
```

Creates Kubernetes DNS:

```text
opentelemetry-demo-recommendationservice
```

Applications communicate using:

```text
opentelemetry-demo-recommendationservice:8080
```

---

# Labels Section

## Label 1

```yaml
opentelemetry.io/name: opentelemetry-demo-recommendationservice
```

Identifies the RecommendationService component.

Useful for resource selection.

---

## Label 2

```yaml
app.kubernetes.io/instance: opentelemetry-demo
```

Indicates this resource belongs to the OpenTelemetry Demo deployment.

---

## Label 3

```yaml
app.kubernetes.io/component: recommendationservice
```

One of the most important labels.

Defines:

```text
Component = recommendationservice
```

Used for:

* Monitoring
* Logging
* Dashboards
* Service grouping

---

## Label 4

```yaml
app.kubernetes.io/name: opentelemetry-demo-recommendationservice
```

Standard Kubernetes naming label.

---

## Label 5

```yaml
app.kubernetes.io/version: "1.12.0"
```

Specifies deployed application version.

---

## Label 6

```yaml
app.kubernetes.io/part-of: opentelemetry-demo
```

Indicates the service belongs to the OpenTelemetry Demo application.

---

# Service Type

```yaml
type: ClusterIP
```

Meaning:

```text
Internal Service Only
```

RecommendationService is never accessed directly by end users.

Instead:

```text
Browser
   │
   ▼
Frontend
   │
   ▼
RecommendationService
```

---

# Port Configuration

```yaml
ports:
  - port: 8080
    name: tcp-service
    targetPort: 8080
```

Traffic Flow:

```text
Frontend
    │
    ▼
RecommendationService:8080
    │
    ▼
RecommendationService Pod:8080
```

---

# Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-recommendationservice
```

Matches Pods containing:

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-recommendationservice
```

Traffic is routed to those Pods.

---

# Chapter 18: Service – opentelemetry-demo-shippingservice

The next Kubernetes Service is:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: opentelemetry-demo-shippingservice
  labels:
    opentelemetry.io/name: opentelemetry-demo-shippingservice
    app.kubernetes.io/instance: opentelemetry-demo
    app.kubernetes.io/component: shippingservice
    app.kubernetes.io/name: opentelemetry-demo-shippingservice
    app.kubernetes.io/version: "1.12.0"
    app.kubernetes.io/part-of: opentelemetry-demo
spec:
  type: ClusterIP
  ports:
    - port: 8080
      name: tcp-service
      targetPort: 8080
  selector:
    opentelemetry.io/name: opentelemetry-demo-shippingservice
```

---

# What is ShippingService?

ShippingService is responsible for handling product shipment operations after a customer places an order.

Once payment is successful, CheckoutService contacts ShippingService to calculate shipping details and create shipment information.

---

# Where is ShippingService Used?

During checkout:

```text
User
 │
 ▼
Frontend
 │
 ▼
CheckoutService
 │
 ▼
ShippingService
```

ShippingService helps determine:

* Shipping cost
* Delivery options
* Shipping method
* Shipment confirmation

---

# Example Checkout Flow

```text
Customer Places Order
         │
         ▼
CheckoutService
         │
         ├── PaymentService
         ├── CurrencyService
         └── ShippingService
                  │
                  ▼
            Create Shipment
```

---

# Why Is ShippingService Important?

Without ShippingService:

```text
Payment Successful
      ✓

Order Created
      ✓

Shipment Created
      ✗
```

The customer would purchase the product, but delivery information could not be generated.

---

# Metadata Section

## Name

```yaml
name: opentelemetry-demo-shippingservice
```

Creates a DNS entry:

```text
opentelemetry-demo-shippingservice
```

Applications connect using:

```text
opentelemetry-demo-shippingservice:8080
```

---

# Labels Section

## Label 1

```yaml
opentelemetry.io/name: opentelemetry-demo-shippingservice
```

Identifies this resource as the ShippingService component.

Useful for resource filtering:

```bash
kubectl get all -l opentelemetry.io/name=opentelemetry-demo-shippingservice
```

---

## Label 2

```yaml
app.kubernetes.io/instance: opentelemetry-demo
```

Identifies the application instance.

Meaning:

```text
This ShippingService belongs to
the OpenTelemetry Demo deployment.
```

---

## Label 3

```yaml
app.kubernetes.io/component: shippingservice
```

Very important label.

Defines:

```text
Component = shippingservice
```

Used by:

* Monitoring systems
* Dashboards
* Logging tools
* Resource grouping

---

## Label 4

```yaml
app.kubernetes.io/name: opentelemetry-demo-shippingservice
```

Standard Kubernetes application naming label.

---

## Label 5

```yaml
app.kubernetes.io/version: "1.12.0"
```

Specifies application version.

Useful during:

* Upgrades
* Rollbacks
* Troubleshooting

---

## Label 6

```yaml
app.kubernetes.io/part-of: opentelemetry-demo
```

Shows this service belongs to the OpenTelemetry Demo application.

---

# Service Type

```yaml
type: ClusterIP
```

Meaning:

```text
Internal Kubernetes Service
```

Only services inside the cluster can access it.

---

# Port Configuration

```yaml
ports:
  - port: 8080
    name: tcp-service
    targetPort: 8080
```

Traffic Flow:

```text
CheckoutService
       │
       ▼
ShippingService:8080
       │
       ▼
ShippingService Pod:8080
```

---

# Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-shippingservice
```

Kubernetes finds Pods containing:

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-shippingservice
```

and routes traffic to them.

---

# Chapter 19: Service – opentelemetry-demo-valkey

The next Kubernetes Service is:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: opentelemetry-demo-valkey
  labels:
    opentelemetry.io/name: opentelemetry-demo-valkey
    app.kubernetes.io/instance: opentelemetry-demo
    app.kubernetes.io/component: valkey
    app.kubernetes.io/name: opentelemetry-demo-valkey
    app.kubernetes.io/version: "1.12.0"
    app.kubernetes.io/part-of: opentelemetry-demo
spec:
  type: ClusterIP
  ports:
    - port: 6379
      name: valkey
      targetPort: 6379
  selector:
    opentelemetry.io/name: opentelemetry-demo-valkey
```

---

# What is Valkey?

Valkey is an in-memory key-value database.

It is a community-driven fork of Redis and is compatible with Redis protocols.

Valkey stores data in memory (RAM), making it extremely fast.

---

# Why Do Applications Use Valkey?

Applications use Valkey for:

* Caching
* Session storage
* Temporary data
* Fast data retrieval
* Message queues

---

# Example Without Valkey

```text
Application
      │
      ▼
Database Query
      │
      ▼
Response
```

Every request must access the database.

This can be slow.

---

# Example With Valkey

```text
Application
      │
      ▼
Valkey Cache
      │
      ▼
Response
```

Frequently accessed data is returned much faster.

---

# Metadata Section

## Name

```yaml
name: opentelemetry-demo-valkey
```

Creates Kubernetes DNS:

```text
opentelemetry-demo-valkey
```

Applications connect using:

```text
opentelemetry-demo-valkey:6379
```

---

# Labels Section

## Label 1

```yaml
opentelemetry.io/name: opentelemetry-demo-valkey
```

Identifies the Valkey component.

Useful for:

```bash
kubectl get all -l opentelemetry.io/name=opentelemetry-demo-valkey
```

---

## Label 2

```yaml
app.kubernetes.io/instance: opentelemetry-demo
```

Indicates this resource belongs to the OpenTelemetry Demo deployment.

---

## Label 3

```yaml
app.kubernetes.io/component: valkey
```

Very important label.

Defines:

```text
Component = valkey
```

Used by:

* Monitoring systems
* Logging systems
* Dashboards
* Operations teams

---

## Label 4

```yaml
app.kubernetes.io/name: opentelemetry-demo-valkey
```

Standard Kubernetes naming label.

---

## Label 5

```yaml
app.kubernetes.io/version: "1.12.0"
```

Specifies deployed version.

Useful during upgrades and maintenance.

---

## Label 6

```yaml
app.kubernetes.io/part-of: opentelemetry-demo
```

Shows Valkey belongs to the OpenTelemetry Demo application.

---

# Service Type

```yaml
type: ClusterIP
```

Meaning:

```text
Internal Service Only
```

Valkey should never be directly exposed to internet users.

Only application services access it.

---

# Port Configuration

```yaml
ports:
  - port: 6379
    name: valkey
    targetPort: 6379
```

### Understanding Port 6379

```text
6379
```

is the default Valkey/Redis port.

Most Redis-compatible clients automatically expect this port.

---

# Traffic Flow

```text
Application
     │
     ▼
Valkey Service:6379
     │
     ▼
Valkey Pod:6379
```

---

# Selector

```yaml
selector:
  opentelemetry.io/name: opentelemetry-demo-valkey
```

Matches Pods having:

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-valkey
```

Traffic is routed to those Pods.

---

# Why Is Valkey Different from Other Services?

Most services we've documented so far are:

```text
CartService
CheckoutService
PaymentService
ShippingService
RecommendationService
```

These contain business logic.

Valkey is different.

It acts as infrastructure rather than business logic.

```text
Business Services
        │
        ▼
     Valkey
        │
        ▼
 Fast Data Access
```

# Chapter 20: Deployment – opentelemetry-demo-accountingservice

## Introduction

Until now, we have documented Service resources.

A Service only provides:

* Stable DNS name
* Service discovery
* Load balancing
* Network access to Pods

A Service **does not create Pods**.

The actual Pods are created by a Deployment.

Think of it like this:

```text
Deployment
    │
    ▼
Creates and manages
    │
    ▼
Pods
    │
    ▼
Containers
```

---

## Why Do We Need a Deployment?

Suppose we manually create a Pod:

```text
accountingservice-pod
```

If the Pod crashes:

```text
Pod Deleted
```

Application becomes unavailable.

A Deployment solves this problem.

It continuously ensures the required Pods are running.

```text
Deployment
      │
      ▼
1 Pod Required
      │
      ▼
Pod Crashes
      │
      ▼
Deployment Creates New Pod
```

---

## What Does AccountingService Do?

AccountingService is responsible for processing accounting-related events generated by the application.

In this demo it mainly consumes events from Kafka and generates telemetry data.

Flow:

```text
Kafka
  │
  ▼
AccountingService
  │
  ▼
Metrics
Logs
Traces
```

# YAML Breakdown

---

# apiVersion

```yaml
apiVersion: apps/v1
```

## Purpose

Defines which Kubernetes API manages this resource.

For Deployments:

```yaml
apps/v1
```

is the standard API version.

---

# kind

```yaml
kind: Deployment
```

Tells Kubernetes:

```text
Create a Deployment resource
```

Not:

* Pod
* Service
* ConfigMap
* Secret

but specifically a Deployment.

---

# metadata Section

Contains identifying information about the Deployment.

---

## Name

```yaml
name: opentelemetry-demo-accountingservice
```

Deployment name.

Useful commands:

```bash
kubectl get deployment
kubectl describe deployment opentelemetry-demo-accountingservice
```

---

# Labels

These labels identify the Deployment.

---

## Label 1

```yaml
opentelemetry.io/name: opentelemetry-demo-accountingservice
```

Primary application label.

Used for selecting resources.

---

## Label 2

```yaml
app.kubernetes.io/instance: opentelemetry-demo
```

Application instance name.

---

## Label 3

```yaml
app.kubernetes.io/component: accountingservice
```

Defines:

```text
Component = accountingservice
```

Very important because it identifies the business role.

---

## Label 4

```yaml
app.kubernetes.io/name: opentelemetry-demo-accountingservice
```

Application name.

---

## Label 5

```yaml
app.kubernetes.io/version: "1.12.0"
```

Current deployed version.

---

## Label 6

```yaml
app.kubernetes.io/part-of: opentelemetry-demo
```

Shows this component belongs to the OpenTelemetry Demo application.

---

# spec Section

The spec section defines how Kubernetes should run the Deployment.

---

# Replicas

```yaml
replicas: 1
```

## Meaning

Run exactly one Pod.

```text
Deployment
     │
     ▼
1 AccountingService Pod
```

---

## If Pod Crashes

Kubernetes automatically recreates it.

```text
Pod Deleted
     │
     ▼
Deployment Detects Issue
     │
     ▼
New Pod Created
```

---

# revisionHistoryLimit

```yaml
revisionHistoryLimit: 10
```

## Purpose

Kubernetes stores old Deployment versions.

Example:

```text
Version 1
Version 2
Version 3
```

This setting tells Kubernetes:

```text
Keep last 10 revisions
```

---

## Why Useful?

Allows rollback.

Example:

```bash
kubectl rollout undo deployment/opentelemetry-demo-accountingservice
```

---

# Selector Section

```yaml
selector:
  matchLabels:
```

Very important.

The Deployment uses this selector to identify Pods it owns.

---

## Match Label

```yaml
opentelemetry.io/name: opentelemetry-demo-accountingservice
```

Deployment manages Pods having this label.

---

# Pod Template Section

```yaml
template:
```

This section describes the Pod Kubernetes should create.

Think of it as:

```text
Pod Blueprint
```

Whenever a new Pod is needed, Kubernetes uses this template.

---

# Template Metadata

```yaml
template:
  metadata:
    labels:
```

Labels attached directly to Pods.

---

## Why Important?

The selector must match these labels.

Selector:

```yaml
opentelemetry.io/name: opentelemetry-demo-accountingservice
```

Pod Label:

```yaml
opentelemetry.io/name: opentelemetry-demo-accountingservice
```

Match = Success

If they don't match:

```text
Deployment cannot manage Pods
```

---

# Pod Spec

```yaml
spec:
```

Defines what runs inside the Pod.

---

# Service Account

```yaml
serviceAccountName: opentelemetry-demo
```

The Pod runs using the ServiceAccount we documented earlier.

```text
Pod
 │
 ▼
Uses ServiceAccount
opentelemetry-demo
```

---

# Containers Section

```yaml
containers:
```

Defines application containers running inside the Pod.

This is the most important section of the Deployment.

---

# Container Name

```yaml
name: accountingservice
```

Container identifier.

Useful commands:

```bash
kubectl logs <pod-name> -c accountingservice
```

---

# Container Image

```yaml
image: ghcr.io/open-telemetry/demo:1.12.0-accountingservice
```

This is the container image downloaded from GitHub Container Registry.

Structure:

```text
ghcr.io
   │
Repository
   │
open-telemetry/demo
   │
Tag
   │
1.12.0-accountingservice
```

---

# Image Pull Policy

```yaml
imagePullPolicy: IfNotPresent
```

Meaning:

```text
If image exists locally
    │
    ▼
Use it

Otherwise
    │
    ▼
Download image
```

---

# Environment Variables

```yaml
env:
```

These provide configuration to the container.

---

# OTEL_SERVICE_NAME

```yaml
- name: OTEL_SERVICE_NAME
```

Used by OpenTelemetry.

---

## valueFrom

```yaml
valueFrom:
```

Instead of hardcoding a value, Kubernetes dynamically retrieves it.

---

## fieldRef

```yaml
fieldRef:
```

Reads information from the running Pod.

---

## fieldPath

```yaml
fieldPath: metadata.labels['app.kubernetes.io/component']
```

Reads:

```yaml
app.kubernetes.io/component: accountingservice
```

Result:

```text
OTEL_SERVICE_NAME=accountingservice
```

---

# OTEL_COLLECTOR_NAME

```yaml
value: opentelemetry-demo-otelcol
```

Collector Service Name.

AccountingService sends telemetry here.

---

# OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE

```yaml
value: cumulative
```

Controls metric aggregation behavior.

---

# KAFKA_SERVICE_ADDR

```yaml
value: opentelemetry-demo-kafka:9092
```

Kafka connection address.

AccountingService connects to:

```text
opentelemetry-demo-kafka
```

on port:

```text
9092
```

---

# OTEL_EXPORTER_OTLP_ENDPOINT

```yaml
value: http://$(OTEL_COLLECTOR_NAME):4318
```

Expands to:

```text
http://opentelemetry-demo-otelcol:4318
```

Telemetry destination.

---

# OTEL_RESOURCE_ATTRIBUTES

```yaml
service.name=$(OTEL_SERVICE_NAME)
service.namespace=opentelemetry-demo
service.version=1.12.0
```

Metadata attached to traces, logs, and metrics.

Example:

```text
Service Name = accountingservice
Namespace = opentelemetry-demo
Version = 1.12.0
```

---

# Resources Section

```yaml
resources:
```

Defines container resource limits.

---

# Memory Limit

```yaml
limits:
  memory: 120Mi
```

Maximum memory allowed:

```text
120 MiB
```

If exceeded:

```text
Container may be terminated
(OOMKilled)
```

---

# volumeMounts

```yaml
volumeMounts:
```

Currently empty.

Meaning:

```text
No volumes mounted
```

at this stage.

---

# Init Containers

```yaml
initContainers:
```

Very important section.

Init containers run before the main application starts.

---

# Purpose

The accounting service depends on Kafka.

If Kafka isn't ready:

```text
AccountingService Starts
      │
      ▼
Connection Failure
```

To avoid this:

```text
Wait For Kafka
      │
      ▼
Start AccountingService
```

---

# Init Container Name

```yaml
name: wait-for-kafka
```

This container waits for Kafka.

---

# Image

```yaml
image: busybox:latest
```

A lightweight Linux image.

---

# Command

```yaml
sh -c
```

Runs a shell script.

---

# Script

```bash
until nc -z -v -w30 opentelemetry-demo-kafka 9092
do
  echo waiting for kafka
  sleep 2
done
```

### What Happens?

Step 1:

```text
Try connecting to Kafka
```

Step 2:

```text
Connection Failed
```

Step 3:

```text
Wait 2 seconds
```

Step 4:

```text
Try Again
```

Loop continues until:

```text
Kafka Ready
```

Then:

```text
Main Container Starts
```

---

# Volumes

```yaml
volumes:
```

Currently empty.

No persistent or shared storage is configured.

---

# Complete Deployment Flow

```text
Deployment Created
        │
        ▼
Init Container Starts
        │
        ▼
Check Kafka Availability
        │
        ▼
Kafka Ready
        │
        ▼
AccountingService Container Starts
        │
        ▼
Connect To Kafka
        │
        ▼
Generate Telemetry
        │
        ▼
Send Data To OTEL Collector
```

# Deployment Documentation – AdService

## 1. Purpose of AdService

AdService is responsible for providing advertisements that appear on the frontend product pages.

When a user visits the store:

* Frontend requests ads from AdService.
* AdService generates ad recommendations.
* Ads are displayed on the website.
* Service telemetry is sent to OpenTelemetry Collector.
* Feature flags are obtained from Flagd.

---

# Complete Deployment Overview

| Property        | Value                                        |
| --------------- | -------------------------------------------- |
| Deployment Name | opentelemetry-demo-adservice                 |
| Container Name  | adservice                                    |
| Image           | ghcr.io/open-telemetry/demo:1.12.0-adservice |
| Replicas        | 1                                            |
| Service Account | opentelemetry-demo                           |
| Container Port  | 8080                                         |
| Memory Limit    | 300Mi                                        |

---

# 2. Metadata Section

```yaml
metadata:
  name: opentelemetry-demo-adservice
```

### Explanation

Deployment name.

Kubernetes creates deployment using this name.

```yaml
opentelemetry-demo-adservice
```

This deployment manages AdService pods.

---

# 3. Labels Section

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-adservice
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: adservice
  app.kubernetes.io/name: opentelemetry-demo-adservice
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

## Label Breakdown

### opentelemetry.io/name

```yaml
opentelemetry-demo-adservice
```

Unique identifier for AdService.

Used by Services and Deployments to find pods.

---

### app.kubernetes.io/instance

```yaml
opentelemetry-demo
```

Indicates this deployment belongs to OpenTelemetry Demo application.

---

### app.kubernetes.io/component

```yaml
adservice
```

Specifies component name.

This label is later used for:

```yaml
OTEL_SERVICE_NAME
```

---

### app.kubernetes.io/name

```yaml
opentelemetry-demo-adservice
```

Application name.

---

### app.kubernetes.io/version

```yaml
1.12.0
```

Application version.

---

### app.kubernetes.io/part-of

```yaml
opentelemetry-demo
```

Shows AdService is part of larger OpenTelemetry Demo system.

---

# 4. Deployment Specification

```yaml
spec:
  replicas: 1
```

### replicas

Only one AdService pod will run.

---

```yaml
revisionHistoryLimit: 10
```

Kubernetes stores last 10 deployment revisions.

Useful for rollback.

---

# 5. Selector

```yaml
selector:
  matchLabels:
    opentelemetry.io/name: opentelemetry-demo-adservice
```

### Purpose

Deployment controls pods having this label.

Only matching pods are managed.

---

# 6. Pod Template

```yaml
template:
```

Defines blueprint for future pods.

Whenever Kubernetes creates a pod, it uses this template.

---

## Pod Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-adservice
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: adservice
  app.kubernetes.io/name: opentelemetry-demo-adservice
```

These labels are attached directly to pods.

---

# 7. Service Account

```yaml
serviceAccountName: opentelemetry-demo
```

Pod runs using previously created ServiceAccount.

Provides Kubernetes identity.

---

# 8. Container Configuration

```yaml
containers:
  - name: adservice
```

Container name inside pod.

---

## Container Image

```yaml
image: ghcr.io/open-telemetry/demo:1.12.0-adservice
```

Image containing AdService application.

---

## Image Pull Policy

```yaml
imagePullPolicy: IfNotPresent
```

Behavior:

* Pull image if not available locally.
* Use local image if already downloaded.

---

# 9. Container Port

```yaml
ports:
  - containerPort: 8080
    name: service
```

AdService listens on:

```text
8080
```

for incoming requests.

---

# 10. Environment Variables

## OTEL_SERVICE_NAME

```yaml
valueFrom:
  fieldRef:
    fieldPath: metadata.labels['app.kubernetes.io/component']
```

Automatically reads:

```text
adservice
```

Used as telemetry service name.

---

## OTEL_COLLECTOR_NAME

```yaml
opentelemetry-demo-otelcol
```

OpenTelemetry Collector hostname.

---

## OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE

```yaml
cumulative
```

Metrics are exported cumulatively.

---

## AD_SERVICE_PORT

```yaml
8080
```

Port on which AdService runs.

---

## FLAGD_HOST

```yaml
opentelemetry-demo-flagd
```

Feature flag service hostname.

---

## FLAGD_PORT

```yaml
8013
```

Feature flag service port.

---

## OTEL_EXPORTER_OTLP_ENDPOINT

```yaml
http://$(OTEL_COLLECTOR_NAME):4318
```

Expands to:

```text
http://opentelemetry-demo-otelcol:4318
```

Telemetry data sent here.

---

## OTEL_LOGS_EXPORTER

```yaml
otlp
```

Logs exported using OTLP protocol.

---

## OTEL_RESOURCE_ATTRIBUTES

```yaml
service.name=$(OTEL_SERVICE_NAME),
service.namespace=opentelemetry-demo,
service.version=1.12.0
```

Adds metadata to traces and metrics.

---

# 11. Resource Limits

```yaml
resources:
  limits:
    memory: 300Mi
```

Maximum memory allowed:

```text
300 MB
```

If exceeded, pod may be terminated.

---

# 12. Volume Mounts

```yaml
volumeMounts:
```

Currently empty.

No external storage mounted.

---

# 13. Volumes

```yaml
volumes:
```

No volumes defined.

---

# Deployment Documentation – CartService

## 1. Purpose of CartService

CartService manages the user's shopping cart.

Functions:

* Add products to cart
* Remove products from cart
* Update quantity
* Retrieve cart details
* Store cart data in Valkey (Redis-compatible cache)

---

# Complete Deployment Overview

| Property        | Value                                          |
| --------------- | ---------------------------------------------- |
| Deployment Name | opentelemetry-demo-cartservice                 |
| Container Name  | cartservice                                    |
| Image           | ghcr.io/open-telemetry/demo:1.12.0-cartservice |
| Port            | 8080                                           |
| Memory Limit    | 160Mi                                          |
| Database        | Valkey                                         |
| Feature Flags   | Flagd                                          |

---

# 2. Metadata

```yaml
metadata:
  name: opentelemetry-demo-cartservice
```

Deployment name.

---

# 3. Labels

```yaml
opentelemetry.io/name: opentelemetry-demo-cartservice
app.kubernetes.io/instance: opentelemetry-demo
app.kubernetes.io/component: cartservice
app.kubernetes.io/name: opentelemetry-demo-cartservice
app.kubernetes.io/version: "1.12.0"
app.kubernetes.io/part-of: opentelemetry-demo
```

## Purpose of Labels

| Label                 | Purpose              |
| --------------------- | -------------------- |
| opentelemetry.io/name | Pod identification   |
| instance              | Application instance |
| component             | Component name       |
| name                  | Resource name        |
| version               | Release version      |
| part-of               | Parent application   |

---

# 4. Deployment Configuration

## Replicas

```yaml
replicas: 1
```

One CartService pod runs.

---

## Revision History

```yaml
revisionHistoryLimit: 10
```

Stores 10 previous versions.

---

# 5. Selector

```yaml
selector:
  matchLabels:
    opentelemetry.io/name: opentelemetry-demo-cartservice
```

Deployment manages pods having this label.

---

# 6. Service Account

```yaml
serviceAccountName: opentelemetry-demo
```

Pod identity.

---

# 7. Container

```yaml
name: cartservice
```

Container name.

---

## Image

```yaml
ghcr.io/open-telemetry/demo:1.12.0-cartservice
```

CartService application image.

---

## Port

```yaml
containerPort: 8080
```

Application listens on port 8080.

---

# 8. Environment Variables

## CART_SERVICE_PORT

```yaml
8080
```

Application port.

---

## ASPNETCORE_URLS

```yaml
http://*:$(CART_SERVICE_PORT)
```

Expands to:

```text
http://*:8080
```

.NET application listens on all interfaces.

---

## VALKEY_ADDR

```yaml
opentelemetry-demo-valkey:6379
```

Valkey server address.

Cart data is stored here.

---

## FLAGD_HOST

```yaml
opentelemetry-demo-flagd
```

Feature flag service.

---

## FLAGD_PORT

```yaml
8013
```

Feature flag port.

---

## OTEL_EXPORTER_OTLP_ENDPOINT

```yaml
http://opentelemetry-demo-otelcol:4317
```

Telemetry destination.

---

## OTEL_RESOURCE_ATTRIBUTES

Adds:

* service name
* namespace
* version

to telemetry.

---

# 9. Resource Limits

```yaml
memory: 160Mi
```

Maximum memory:

```text
160 MB
```

---

# 10. Init Container

```yaml
initContainers:
```

Before CartService starts:

```yaml
wait-for-valkey
```

runs.

---

## Command

```sh
until nc -z -v -w30 opentelemetry-demo-valkey 6379
```

Meaning:

```text
Check if Valkey is available.
If not available:
wait 2 seconds
try again
```

---

### Why Needed?

Without Valkey:

```text
CartService cannot store cart data.
```

So Kubernetes waits.

---

# 11. Volumes

```yaml
volumes:
```

No volumes configured.

---

# Deployment Documentation – CheckoutService

## 1. Purpose of CheckoutService

CheckoutService is one of the most important services in the OpenTelemetry Demo application.

When a user clicks **"Place Order"**, CheckoutService coordinates the entire order process.

### Responsibilities

* Retrieves cart information
* Gets product details
* Calculates prices
* Converts currency
* Processes payment
* Calculates shipping cost
* Sends confirmation email
* Publishes order events to Kafka
* Exports telemetry data

Think of CheckoutService as the **orchestrator** of the e-commerce application.

---

# Complete Deployment Overview

| Property             | Value                                              |
| -------------------- | -------------------------------------------------- |
| Deployment Name      | opentelemetry-demo-checkoutservice                 |
| Container Name       | checkoutservice                                    |
| Image                | ghcr.io/open-telemetry/demo:1.12.0-checkoutservice |
| Replicas             | 1                                                  |
| Port                 | 8080                                               |
| Memory Limit         | 20Mi                                               |
| Kafka Dependency     | Yes                                                |
| Feature Flag Support | Yes                                                |

---

# 2. Metadata Section

```yaml
metadata:
  name: opentelemetry-demo-checkoutservice
```

### Purpose

Defines the deployment name.

Kubernetes uses this name to manage CheckoutService pods.

---

# 3. Labels Section

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-checkoutservice
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: checkoutservice
  app.kubernetes.io/name: opentelemetry-demo-checkoutservice
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

## Label Explanation

### opentelemetry.io/name

Unique label used for:

* Pod identification
* Service discovery
* Deployment selector matching

---

### app.kubernetes.io/instance

```yaml
opentelemetry-demo
```

Identifies the application instance.

---

### app.kubernetes.io/component

```yaml
checkoutservice
```

Identifies the component type.

Used later to populate:

```yaml
OTEL_SERVICE_NAME
```

---

### app.kubernetes.io/version

```yaml
1.12.0
```

Application version.

---

### app.kubernetes.io/part-of

```yaml
opentelemetry-demo
```

Shows this service belongs to the OpenTelemetry Demo application.

---

# 4. Deployment Configuration

## Replicas

```yaml
replicas: 1
```

Only one CheckoutService pod is created.

---

## Revision History

```yaml
revisionHistoryLimit: 10
```

Stores 10 previous deployment revisions.

Useful for rollback.

---

# 5. Selector

```yaml
selector:
  matchLabels:
    opentelemetry.io/name: opentelemetry-demo-checkoutservice
```

Deployment controls pods containing this label.

---

# 6. Pod Template

Defines how Kubernetes creates CheckoutService pods.

---

# 7. Service Account

```yaml
serviceAccountName: opentelemetry-demo
```

Pod runs using the ServiceAccount created earlier.

Provides Kubernetes identity.

---

# 8. Container Configuration

## Container Name

```yaml
name: checkoutservice
```

Main application container.

---

## Image

```yaml
image: ghcr.io/open-telemetry/demo:1.12.0-checkoutservice
```

Docker image containing CheckoutService application.

---

## Port

```yaml
containerPort: 8080
```

Application listens on port 8080.

---

# 9. Environment Variables

## CHECKOUT_SERVICE_PORT

```yaml
8080
```

Port used by CheckoutService.

---

## CART_SERVICE_ADDR

```yaml
opentelemetry-demo-cartservice:8080
```

Used to retrieve cart contents.

---

## CURRENCY_SERVICE_ADDR

```yaml
opentelemetry-demo-currencyservice:8080
```

Used for currency conversion.

---

## EMAIL_SERVICE_ADDR

```yaml
http://opentelemetry-demo-emailservice:8080
```

Used to send order confirmation emails.

---

## PAYMENT_SERVICE_ADDR

```yaml
opentelemetry-demo-paymentservice:8080
```

Used to process payments.

---

## PRODUCT_CATALOG_SERVICE_ADDR

```yaml
opentelemetry-demo-productcatalogservice:8080
```

Used to fetch product details.

---

## SHIPPING_SERVICE_ADDR

```yaml
opentelemetry-demo-shippingservice:8080
```

Used to calculate shipping charges.

---

## KAFKA_SERVICE_ADDR

```yaml
opentelemetry-demo-kafka:9092
```

Publishes order events to Kafka.

---

## FLAGD_HOST

```yaml
opentelemetry-demo-flagd
```

Feature flag service hostname.

---

## FLAGD_PORT

```yaml
8013
```

Feature flag service port.

---

## OTEL_EXPORTER_OTLP_ENDPOINT

```yaml
http://opentelemetry-demo-otelcol:4317
```

Telemetry destination.

---

## OTEL_RESOURCE_ATTRIBUTES

Adds:

* service name
* namespace
* version

to telemetry data.

---

# 10. Resources

```yaml
resources:
  limits:
    memory: 20Mi
```

Maximum memory allowed:

```text
20 MB
```

Very lightweight service.

---

# 11. Init Container

## Why Init Container Exists

CheckoutService depends on Kafka.

If Kafka is unavailable, order processing may fail.

Therefore Kubernetes waits until Kafka is ready.

---

## Init Container Name

```yaml
wait-for-kafka
```

---

## Command

```sh
until nc -z -v -w30 opentelemetry-demo-kafka 9092
```

Meaning:

```text
Check Kafka availability
If unavailable:
wait 2 seconds
retry
```

Only after Kafka becomes reachable will CheckoutService start.

---

# 12. Volumes

```yaml
volumes:
```

No volumes configured.

---

# 13. Service Dependencies

| Service               | Purpose             |
| --------------------- | ------------------- |
| CartService           | Retrieve cart       |
| CurrencyService       | Currency conversion |
| EmailService          | Send emails         |
| PaymentService        | Payment processing  |
| ProductCatalogService | Product details     |
| ShippingService       | Shipping cost       |
| Kafka                 | Event streaming     |
| Flagd                 | Feature flags       |
| OTEL Collector        | Telemetry           |

---

# 14. Runtime Flow

```text
User
 ↓
Frontend
 ↓
CheckoutService
 ├── CartService
 ├── ProductCatalogService
 ├── PaymentService
 ├── CurrencyService
 ├── ShippingService
 ├── EmailService
 └── Kafka
```

CheckoutService acts as the central coordinator.


# Deployment Documentation – CurrencyService

## 1. Purpose of CurrencyService

CurrencyService is responsible for currency conversion.

### Example

If product price is:

```text
$100 USD
```

and the user selects:

```text
INR
```

CurrencyService converts the value before it is displayed.

---

# Complete Deployment Overview

| Property         | Value                                              |
| ---------------- | -------------------------------------------------- |
| Deployment Name  | opentelemetry-demo-currencyservice                 |
| Container Name   | currencyservice                                    |
| Image            | ghcr.io/open-telemetry/demo:1.12.0-currencyservice |
| Port             | 8080                                               |
| Memory Limit     | 20Mi                                               |
| Version Variable | Yes                                                |

---

# 2. Metadata

```yaml
metadata:
  name: opentelemetry-demo-currencyservice
```

Unique deployment name.

---

# 3. Labels

```yaml
opentelemetry.io/name: opentelemetry-demo-currencyservice
app.kubernetes.io/instance: opentelemetry-demo
app.kubernetes.io/component: currencyservice
app.kubernetes.io/name: opentelemetry-demo-currencyservice
app.kubernetes.io/version: "1.12.0"
app.kubernetes.io/part-of: opentelemetry-demo
```

Used for:

* Identification
* Service selection
* Monitoring
* Version tracking

---

# 4. Deployment Configuration

## Replicas

```yaml
replicas: 1
```

Runs a single pod.

---

## Revision History

```yaml
revisionHistoryLimit: 10
```

Stores previous deployment versions.

---

# 5. Container Configuration

## Name

```yaml
currencyservice
```

---

## Image

```yaml
ghcr.io/open-telemetry/demo:1.12.0-currencyservice
```

Currency conversion application image.

---

## Port

```yaml
containerPort: 8080
```

Application listens on port 8080.

---

# 6. Environment Variables

## CURRENCY_SERVICE_PORT

```yaml
8080
```

Application listening port.

---

## VERSION

```yaml
1.12.0
```

Application version.

Useful for:

* debugging
* telemetry
* version visibility

---

## OTEL_COLLECTOR_NAME

```yaml
opentelemetry-demo-otelcol
```

Collector hostname.

---

## OTEL_EXPORTER_OTLP_ENDPOINT

```yaml
http://opentelemetry-demo-otelcol:4317
```

Telemetry destination.

---

## OTEL_RESOURCE_ATTRIBUTES

Adds service metadata into telemetry.

---

# 7. Resources

```yaml
memory: 20Mi
```

Maximum memory usage:

```text
20 MB
```

---

# 8. Volumes

No volumes defined.

---

# 9. Service Dependencies

| Service        | Purpose          |
| -------------- | ---------------- |
| OTEL Collector | Telemetry export |

CurrencyService is largely independent.

---

# 10. Runtime Flow

```text
Frontend
 ↓
CurrencyService
 ↓
Convert Amount
 ↓
Return Result
```

---

# Deployment Documentation – EmailService

## 1. Purpose of EmailService

EmailService sends order confirmation emails after successful purchases.

### Example

After checkout:

```text
Order Successful
       ↓
CheckoutService
       ↓
EmailService
       ↓
Send Confirmation Email
```

---

# Complete Deployment Overview

| Property        | Value                                           |
| --------------- | ----------------------------------------------- |
| Deployment Name | opentelemetry-demo-emailservice                 |
| Container Name  | emailservice                                    |
| Image           | ghcr.io/open-telemetry/demo:1.12.0-emailservice |
| Port            | 8080                                            |
| Memory Limit    | 100Mi                                           |
| Environment     | Production                                      |

---

# 2. Metadata

```yaml
metadata:
  name: opentelemetry-demo-emailservice
```

Deployment name.

---

# 3. Labels

```yaml
opentelemetry.io/name: opentelemetry-demo-emailservice
app.kubernetes.io/instance: opentelemetry-demo
app.kubernetes.io/component: emailservice
app.kubernetes.io/name: opentelemetry-demo-emailservice
app.kubernetes.io/version: "1.12.0"
app.kubernetes.io/part-of: opentelemetry-demo
```

Used for identification, monitoring, and service selection.

---

# 4. Container Configuration

## Container Name

```yaml
emailservice
```

---

## Image

```yaml
ghcr.io/open-telemetry/demo:1.12.0-emailservice
```

Contains EmailService application.

---

## Port

```yaml
containerPort: 8080
```

Service listens on port 8080.

---

# 5. Environment Variables

## EMAIL_SERVICE_PORT

```yaml
8080
```

Application port.

---

## APP_ENV

```yaml
production
```

Runs service in production mode.

---

## OTEL_EXPORTER_OTLP_TRACES_ENDPOINT

```yaml
http://$(OTEL_COLLECTOR_NAME):4318/v1/traces
```

Expanded value:

```text
http://opentelemetry-demo-otelcol:4318/v1/traces
```

Trace data is sent to OpenTelemetry Collector.

---

## OTEL_SERVICE_NAME

Automatically resolved from:

```yaml
metadata.labels['app.kubernetes.io/component']
```

Value becomes:

```text
emailservice
```

---

## OTEL_RESOURCE_ATTRIBUTES

Adds:

* service name
* namespace
* version

to telemetry.

---

# 6. Resources

```yaml
memory: 100Mi
```

Maximum memory usage:

```text
100 MB
```

---

# 7. Volumes

No volumes configured.

---

# 8. Runtime Flow

```text
CheckoutService
        ↓
EmailService
        ↓
Generate Email
        ↓
Export Traces
```

---

# Deployment Documentation – Flagd Deployment

# 1. Purpose of Flagd

Flagd is the **Feature Flag Management Service** in the OpenTelemetry Demo application.

Feature flags allow developers to:

* Enable or disable features without redeploying applications
* Perform A/B testing
* Gradually roll out new functionality
* Simulate different business scenarios

Many services in this project use:

```text
FLAGD_HOST=opentelemetry-demo-flagd
FLAGD_PORT=8013
```

which means they connect to Flagd to retrieve feature flag values.

Examples:

* AdService
* CartService
* CheckoutService
* PaymentService
* ProductCatalogService
* Frontend
* RecommendationService
* FraudDetectionService

All depend on Flagd.

---

# High-Level Architecture

```text
                Flag Configuration
                         ↓
                   ConfigMap
                         ↓
                  Init Container
                         ↓
                    Flagd
                         ↓
     ------------------------------------
     |       |       |        |         |
 AdService Cart  Checkout Frontend Payment
```

---

# 2. Metadata Section

```yaml
metadata:
  name: opentelemetry-demo-flagd
```

## name

```yaml
opentelemetry-demo-flagd
```

Deployment name.

Kubernetes uses this name to:

* Identify deployment
* Manage replicas
* Perform updates
* Perform rollbacks

---

# 3. Labels Section

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-flagd
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: flagd
  app.kubernetes.io/name: opentelemetry-demo-flagd
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

## opentelemetry.io/name

```yaml
opentelemetry-demo-flagd
```

Primary identifier for Flagd pods.

Used by:

* Deployment selector
* Service selector

---

## app.kubernetes.io/instance

```yaml
opentelemetry-demo
```

Application instance name.

Useful when multiple deployments exist.

---

## app.kubernetes.io/component

```yaml
flagd
```

Component name.

Used later for:

```yaml
OTEL_SERVICE_NAME
```

---

## app.kubernetes.io/name

Application resource name.

---

## app.kubernetes.io/version

```yaml
1.12.0
```

Current application version.

---

## app.kubernetes.io/part-of

```yaml
opentelemetry-demo
```

Indicates this deployment belongs to OpenTelemetry Demo.

---

# 4. Deployment Specification

## Replicas

```yaml
replicas: 1
```

Only one Flagd pod is created.

---

## revisionHistoryLimit

```yaml
revisionHistoryLimit: 10
```

Stores previous 10 deployment revisions.

Allows rollback.

---

# 5. Selector Section

```yaml
selector:
  matchLabels:
    opentelemetry.io/name: opentelemetry-demo-flagd
```

Deployment manages only pods matching this label.

Without matching labels:

```text
Deployment cannot control pod
```

---

# 6. Pod Template

```yaml
template:
```

Blueprint used for creating Flagd pods.

---

## Pod Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-flagd
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: flagd
  app.kubernetes.io/name: opentelemetry-demo-flagd
```

Attached to actual pods.

---

# 7. ServiceAccount

```yaml
serviceAccountName: opentelemetry-demo
```

Pod runs using ServiceAccount:

```yaml
opentelemetry-demo
```

Provides Kubernetes identity.

---

# 8. Containers Overview

This deployment is special.

It contains **two containers**.

```text
Flagd Pod
│
├── flagd
│
└── flagdui
```

Most deployments contain only one container.

---

# Container 1 – flagd

## Container Name

```yaml
name: flagd
```

Main feature flag server.

---

## Image

```yaml
image: ghcr.io/open-feature/flagd:v0.11.1
```

Official Flagd image.

---

## Command Section

```yaml
command:
  - /flagd-build
  - start
  - --uri
  - file:./etc/flagd/demo.flagd.json
```

Meaning:

```text
Start Flagd
Read feature flags from
demo.flagd.json file
```

---

## Port

```yaml
containerPort: 8013
```

Feature flag API listens on:

```text
8013
```

---

# Environment Variables

## OTEL_SERVICE_NAME

Automatically gets value:

```text
flagd
```

from pod label.

---

## OTEL_COLLECTOR_NAME

```yaml
opentelemetry-demo-otelcol
```

Collector hostname.

---

## FLAGD_METRICS_EXPORTER

```yaml
otel
```

Export metrics using OpenTelemetry.

---

## FLAGD_OTEL_COLLECTOR_URI

```yaml
$(OTEL_COLLECTOR_NAME):4317
```

Expanded value:

```text
opentelemetry-demo-otelcol:4317
```

Destination for telemetry.

---

## OTEL_RESOURCE_ATTRIBUTES

Adds metadata to telemetry.

---

## Resource Limit

```yaml
memory: 75Mi
```

Maximum memory:

```text
75 MB
```

---

# Container 2 – flagdui

## Purpose

Provides web UI for managing and viewing flags.

Without this container:

```text
No graphical interface
```

---

## Container Name

```yaml
flagdui
```

---

## Image

```yaml
ghcr.io/open-telemetry/demo:1.12.0-flagdui
```

UI application.

---

## Port

```yaml
containerPort: 4000
```

UI accessible on:

```text
4000
```

---

## Environment Variables

### FLAGD_METRICS_EXPORTER

```yaml
otel
```

Exports metrics.

---

### OTEL_EXPORTER_OTLP_ENDPOINT

```yaml
http://opentelemetry-demo-otelcol:4318
```

Telemetry destination.

---

### OTEL_RESOURCE_ATTRIBUTES

Service metadata.

---

## Resource Limit

```yaml
memory: 75Mi
```

Maximum memory allowed.

---

# 9. Init Container

This deployment contains an Init Container.

---

## Why Needed?

Flagd needs:

```text
demo.flagd.json
```

configuration file.

ConfigMap is mounted as read-only.

Flagd needs writable copy.

Therefore InitContainer copies file.

---

## Init Container Name

```yaml
init-config
```

---

## Image

```yaml
busybox
```

Small Linux utility image.

---

## Command

```sh
cp /config-ro/demo.flagd.json /config-rw/demo.flagd.json
```

Meaning:

```text
Copy flag configuration
from read-only location
to writable location
```

---

## Workflow

```text
ConfigMap
     ↓
/config-ro
     ↓
Init Container
     ↓
/config-rw
     ↓
Flagd
```

---

# 10. Volume Mounts

## flagd Container

```yaml
volumeMounts:
  - name: config-rw
    mountPath: /etc/flagd
```

Flagd reads configuration from here.

---

## flagdui Container

```yaml
mountPath: /app/data
```

UI accesses same configuration.

---

# 11. Volumes

## config-rw

```yaml
emptyDir: {}
```

Temporary writable storage.

Created when pod starts.

Deleted when pod is removed.

---

## config-ro

```yaml
configMap:
  name: opentelemetry-demo-flagd-config
```

Read-only configuration source.

Contains:

```text
demo.flagd.json
```

---

# 12. Startup Sequence

```text
Pod Created
      ↓
Init Container Runs
      ↓
Copies Config
      ↓
Flagd Starts
      ↓
Flagd UI Starts
      ↓
Services Connect
```

---

# Deployment Documentation – FraudDetectionService

# 1. Purpose of FraudDetectionService

FraudDetectionService monitors order activity and detects suspicious transactions.

Examples:

* Abnormally large orders
* Suspicious purchasing patterns
* Fraud simulation scenarios
* Event analysis from Kafka

It consumes order-related events from Kafka and performs fraud analysis.

---

# High-Level Architecture

```text
CheckoutService
       ↓
      Kafka
       ↓
FraudDetectionService
       ↓
Fraud Analysis
       ↓
OpenTelemetry Collector
```

---

# 2. Metadata

```yaml
metadata:
  name: opentelemetry-demo-frauddetectionservice
```

Deployment name used by Kubernetes.

---

# 3. Labels

## opentelemetry.io/name

```yaml
opentelemetry-demo-frauddetectionservice
```

Primary pod identifier.

---

## app.kubernetes.io/instance

```yaml
opentelemetry-demo
```

Application instance.

---

## app.kubernetes.io/component

```yaml
frauddetectionservice
```

Component name.

Used to populate:

```yaml
OTEL_SERVICE_NAME
```

---

## app.kubernetes.io/name

Resource name.

---

## app.kubernetes.io/version

```yaml
1.12.0
```

Current version.

---

## app.kubernetes.io/part-of

```yaml
opentelemetry-demo
```

Application grouping label.

---

# 4. Deployment Specification

## Replicas

```yaml
replicas: 1
```

Single fraud detection pod.

---

## Revision History

```yaml
revisionHistoryLimit: 10
```

Stores previous revisions.

---

# 5. Selector

```yaml
selector:
  matchLabels:
    opentelemetry.io/name: opentelemetry-demo-frauddetectionservice
```

Deployment manages pods matching this label.

---

# 6. ServiceAccount

```yaml
serviceAccountName: opentelemetry-demo
```

Pod identity.

---

# 7. Container

## Name

```yaml
frauddetectionservice
```

---

## Image

```yaml
ghcr.io/open-telemetry/demo:1.12.0-frauddetectionservice
```

Contains fraud analysis application.

---

# 8. Environment Variables

## OTEL_SERVICE_NAME

Automatically resolved as:

```text
frauddetectionservice
```

---

## KAFKA_SERVICE_ADDR

```yaml
opentelemetry-demo-kafka:9092
```

Kafka broker address.

Used for consuming order events.

---

## FLAGD_HOST

```yaml
opentelemetry-demo-flagd
```

Feature flag hostname.

---

## FLAGD_PORT

```yaml
8013
```

Feature flag port.

---

## OTEL_EXPORTER_OTLP_ENDPOINT

```yaml
http://opentelemetry-demo-otelcol:4318
```

Telemetry destination.

---

## OTEL_RESOURCE_ATTRIBUTES

Adds:

* service name
* namespace
* version

to telemetry.

---

# 9. Resource Limits

```yaml
memory: 300Mi
```

Maximum memory:

```text
300 MB
```

Higher than many services because fraud analysis requires more processing.

---

# 10. Init Container

## Name

```yaml
wait-for-kafka
```

---

## Purpose

Kafka must be available before FraudDetectionService starts.

---

## Command

```sh
until nc -z -v -w30 opentelemetry-demo-kafka 9092
```

Checks Kafka availability repeatedly.

---

## Startup Flow

```text
Pod Created
      ↓
Check Kafka
      ↓
Kafka Available?
      ↓
     Yes
      ↓
Start Application
```

---

# 11. Volumes

No volumes configured.

---

#  Frontend Deployment Documentation

## Purpose of Frontend Service

The Frontend service is the main web application of the OpenTelemetry Demo. It displays the online store UI that users interact with.

Without Frontend:

* Users cannot browse products.
* Users cannot add items to cart.
* Users cannot place orders.
* Other backend services would run but no customer-facing website would exist.

Flow:

```text
User Browser
      ↓
Frontend
      ↓
------------------------------------
Ad Service
Cart Service
Checkout Service
Currency Service
Product Catalog Service
Recommendation Service
Shipping Service
------------------------------------
```

---

# Complete YAML Structure

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
spec:
```

This creates and manages Frontend pods.

---

# Metadata Section

```yaml
metadata:
  name: opentelemetry-demo-frontend
```

Deployment name.

```yaml
name: opentelemetry-demo-frontend
```

Kubernetes commands:

```bash
kubectl get deployment opentelemetry-demo-frontend
```

---

# Labels Section

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-frontend
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: frontend
  app.kubernetes.io/name: opentelemetry-demo-frontend
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

## Label Explanation

### opentelemetry.io/name

```yaml
opentelemetry.io/name: opentelemetry-demo-frontend
```

Unique identifier for frontend.

---

### app.kubernetes.io/instance

```yaml
app.kubernetes.io/instance: opentelemetry-demo
```

Application instance name.

---

### app.kubernetes.io/component

```yaml
app.kubernetes.io/component: frontend
```

Defines this component as Frontend.

Used later by:

```yaml
OTEL_SERVICE_NAME
```

---

### app.kubernetes.io/name

```yaml
app.kubernetes.io/name: opentelemetry-demo-frontend
```

Human readable component name.

---

### app.kubernetes.io/version

```yaml
app.kubernetes.io/version: "1.12.0"
```

Current application version.

---

### app.kubernetes.io/part-of

```yaml
app.kubernetes.io/part-of: opentelemetry-demo
```

Shows frontend belongs to OpenTelemetry Demo application.

---

# Spec Section

```yaml
spec:
```

Deployment configuration starts here.

---

## Replicas

```yaml
replicas: 1
```

Only one frontend pod runs.

```text
Frontend Pod
```

If pod dies:

```text
Deployment
      ↓
Creates New Pod
```

---

## Revision History

```yaml
revisionHistoryLimit: 10
```

Stores last 10 deployment revisions.

Useful for rollback.

```bash
kubectl rollout undo deployment opentelemetry-demo-frontend
```

---

# Selector

```yaml
selector:
  matchLabels:
    opentelemetry.io/name: opentelemetry-demo-frontend
```

Deployment manages pods having this label.

---

# Pod Template

```yaml
template:
```

Blueprint used to create frontend pods.

---

## Pod Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-frontend
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: frontend
  app.kubernetes.io/name: opentelemetry-demo-frontend
```

Applied to every frontend pod.

---

# Service Account

```yaml
serviceAccountName: opentelemetry-demo
```

Frontend runs using:

```text
ServiceAccount
    ↓
opentelemetry-demo
```

Provides Kubernetes identity.

---

# Container Section

```yaml
containers:
```

One container:

```yaml
name: frontend
```

---

## Container Name

```yaml
name: frontend
```

Container identifier.

---

## Image

```yaml
image: 'ghcr.io/open-telemetry/demo:1.12.0-frontend'
```

Frontend Docker image.

Contains:

* UI code
* HTTP server
* OpenTelemetry instrumentation

---

## Image Pull Policy

```yaml
imagePullPolicy: IfNotPresent
```

Pull image only if missing.

---

# Container Port

```yaml
ports:
  - containerPort: 8080
    name: service
```

Frontend listens on:

```text
8080
```

Traffic flow:

```text
Service
   ↓
Frontend Pod
   ↓
Port 8080
```

---

# Environment Variables

## OTEL_SERVICE_NAME

```yaml
OTEL_SERVICE_NAME
```

Obtained dynamically from:

```yaml
metadata.labels['app.kubernetes.io/component']
```

Value becomes:

```text
frontend
```

---

## OTEL_COLLECTOR_NAME

```yaml
OTEL_COLLECTOR_NAME=opentelemetry-demo-otelcol
```

Collector service name.

---

## FRONTEND_PORT

```yaml
FRONTEND_PORT=8080
```

Application listening port.

---

## FRONTEND_ADDR

```yaml
FRONTEND_ADDR=:8080
```

Bind application to port 8080.

---

## AD_SERVICE_ADDR

```yaml
AD_SERVICE_ADDR=opentelemetry-demo-adservice:8080
```

Frontend contacts Ad Service.

Purpose:

```text
Get advertisements
```

---

## CART_SERVICE_ADDR

```yaml
CART_SERVICE_ADDR=opentelemetry-demo-cartservice:8080
```

Used for:

```text
Add to Cart
View Cart
Update Cart
```

---

## CHECKOUT_SERVICE_ADDR

```yaml
CHECKOUT_SERVICE_ADDR=opentelemetry-demo-checkoutservice:8080
```

Used during checkout process.

---

## CURRENCY_SERVICE_ADDR

```yaml
CURRENCY_SERVICE_ADDR=opentelemetry-demo-currencyservice:8080
```

Used for currency conversion.

Example:

```text
USD → INR
USD → EUR
```

---

## PRODUCT_CATALOG_SERVICE_ADDR

```yaml
PRODUCT_CATALOG_SERVICE_ADDR=opentelemetry-demo-productcatalogservice:8080
```

Fetches products.

Example:

```text
Product List
Product Details
```

---

## RECOMMENDATION_SERVICE_ADDR

```yaml
RECOMMENDATION_SERVICE_ADDR=opentelemetry-demo-recommendationservice:8080
```

Gets recommended products.

---

## SHIPPING_SERVICE_ADDR

```yaml
SHIPPING_SERVICE_ADDR=opentelemetry-demo-shippingservice:8080
```

Calculates shipping costs.

---

## FLAGD_HOST

```yaml
FLAGD_HOST=opentelemetry-demo-flagd
```

Feature flag server.

---

## FLAGD_PORT

```yaml
FLAGD_PORT=8013
```

Feature flag port.

---

## OTEL_COLLECTOR_HOST

```yaml
OTEL_COLLECTOR_HOST=$(OTEL_COLLECTOR_NAME)
```

Resolves to:

```text
opentelemetry-demo-otelcol
```

---

## OTEL_EXPORTER_OTLP_ENDPOINT

```yaml
OTEL_EXPORTER_OTLP_ENDPOINT=http://opentelemetry-demo-otelcol:4317
```

Exports telemetry data.

---

## WEB_OTEL_SERVICE_NAME

```yaml
WEB_OTEL_SERVICE_NAME=frontend-web
```

Browser telemetry service name.

Useful for tracing browser requests separately.

---

## PUBLIC_OTEL_EXPORTER_OTLP_TRACES_ENDPOINT

```yaml
PUBLIC_OTEL_EXPORTER_OTLP_TRACES_ENDPOINT=http://localhost:8080/otlp-http/v1/traces
```

Browser traces sent through frontend proxy.

Flow:

```text
Browser
   ↓
Frontend
   ↓
OTEL Collector
```

---

## OTEL_RESOURCE_ATTRIBUTES

```yaml
service.name=$(OTEL_SERVICE_NAME)
service.namespace=opentelemetry-demo
service.version=1.12.0
```

Adds metadata to telemetry.

---

# Resources

```yaml
resources:
  limits:
    memory: 250Mi
```

Frontend can use:

```text
250 MB RAM
```

Maximum.

---

# Security Context

```yaml
securityContext:
```

Security hardening settings.

---

## runAsGroup

```yaml
runAsGroup: 1001
```

Container group ID.

---

## runAsNonRoot

```yaml
runAsNonRoot: true
```

Cannot run as root.

Security best practice.

---

## runAsUser

```yaml
runAsUser: 1001
```

Runs as user 1001.

Safer than root.

---

# Volume Mounts

```yaml
volumeMounts:
```

No custom volumes attached.

---

# Volumes

```yaml
volumes:
```

No volumes defined.

---

# Frontend Dependency Diagram

```text
                    Frontend
                        │
    ┌──────────┬────────┼─────────┬─────────┐
    │          │        │         │         │
    ▼          ▼        ▼         ▼         ▼
 AdService  Cart   Checkout   Product   Recommendation
                     Service   Catalog     Service
                        │
                        ▼
                  Currency Service

                        │
                        ▼
                 Shipping Service

                        │
                        ▼
                     Flagd

                        │
                        ▼
                OTEL Collector
```

---

# FrontendProxy Deployment Documentation

## Purpose of FrontendProxy Service

FrontendProxy is the external entry point of the entire OpenTelemetry Demo application.

Users do not directly access Frontend.

Instead:

```text
User Browser
      ↓
FrontendProxy
      ↓
Frontend
      ↓
Backend Services
```

FrontendProxy is based on Envoy Proxy and performs:

* Request routing
* Reverse proxy functionality
* OpenTelemetry instrumentation
* Access to Grafana
* Access to Jaeger
* Access to Flagd UI
* Access to Load Generator UI
* Access to Image Provider

Without FrontendProxy:

* Users cannot access application properly
* Routing between UI components fails
* Telemetry proxying becomes difficult

---

# YAML Structure

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
spec:
```

Creates FrontendProxy deployment.

---

# Metadata Section

```yaml
metadata:
  name: opentelemetry-demo-frontendproxy
```

Deployment name.

---

# Labels Section

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-frontendproxy
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: frontendproxy
  app.kubernetes.io/name: opentelemetry-demo-frontendproxy
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

## Label Explanation

### opentelemetry.io/name

```yaml
opentelemetry.io/name: opentelemetry-demo-frontendproxy
```

Unique identifier.

---

### app.kubernetes.io/instance

```yaml
app.kubernetes.io/instance: opentelemetry-demo
```

Application instance.

---

### app.kubernetes.io/component

```yaml
app.kubernetes.io/component: frontendproxy
```

Component type.

Used by:

```yaml
OTEL_SERVICE_NAME
```

Value becomes:

```text
frontendproxy
```

---

### app.kubernetes.io/name

```yaml
app.kubernetes.io/name: opentelemetry-demo-frontendproxy
```

Readable service name.

---

### app.kubernetes.io/version

```yaml
app.kubernetes.io/version: "1.12.0"
```

Application version.

---

### app.kubernetes.io/part-of

```yaml
app.kubernetes.io/part-of: opentelemetry-demo
```

Shows ownership.

---

# Spec Section

## Replicas

```yaml
replicas: 1
```

One FrontendProxy pod.

---

## Revision History

```yaml
revisionHistoryLimit: 10
```

Stores previous deployment revisions.

---

# Selector

```yaml
selector:
  matchLabels:
    opentelemetry.io/name: opentelemetry-demo-frontendproxy
```

Matches FrontendProxy pods.

---

# Pod Template

```yaml
template:
```

Template used for pod creation.

---

# Service Account

```yaml
serviceAccountName: opentelemetry-demo
```

Pod uses OpenTelemetry Demo service account.

---

# Container Section

```yaml
containers:
```

Single container:

```yaml
name: frontendproxy
```

---

# Container Name

```yaml
name: frontendproxy
```

Container identifier.

---

# Docker Image

```yaml
image: ghcr.io/open-telemetry/demo:1.12.0-frontendproxy
```

Contains:

* Envoy Proxy
* Routing rules
* OpenTelemetry instrumentation

---

# Image Pull Policy

```yaml
imagePullPolicy: IfNotPresent
```

Pull image only if absent.

---

# Container Port

```yaml
ports:
  - containerPort: 8080
    name: service
```

FrontendProxy listens on:

```text
8080
```

External traffic enters here.

---

# Environment Variables

---

## OTEL_SERVICE_NAME

```yaml
OTEL_SERVICE_NAME
```

Read dynamically from pod label.

Result:

```text
frontendproxy
```

---

## OTEL_COLLECTOR_NAME

```yaml
OTEL_COLLECTOR_NAME=opentelemetry-demo-otelcol
```

Collector service.

---

## ENVOY_PORT

```yaml
ENVOY_PORT=8080
```

Port used by Envoy proxy.

---

## FLAGD_HOST

```yaml
FLAGD_HOST=opentelemetry-demo-flagd
```

Feature flag server hostname.

---

## FLAGD_PORT

```yaml
FLAGD_PORT=8013
```

Feature flag service port.

---

## FLAGD_UI_HOST

```yaml
FLAGD_UI_HOST=opentelemetry-demo-flagd
```

Flagd UI hostname.

---

## FLAGD_UI_PORT

```yaml
FLAGD_UI_PORT=4000
```

Flagd UI port.

Flow:

```text
Browser
    ↓
FrontendProxy
    ↓
Flagd UI
```

---

## FRONTEND_HOST

```yaml
FRONTEND_HOST=opentelemetry-demo-frontend
```

Actual frontend application.

---

## FRONTEND_PORT

```yaml
FRONTEND_PORT=8080
```

Frontend service port.

Flow:

```text
Browser
    ↓
FrontendProxy
    ↓
Frontend
```

---

## GRAFANA_SERVICE_HOST

```yaml
GRAFANA_SERVICE_HOST=opentelemetry-demo-grafana
```

Grafana service hostname.

Used for dashboards.

---

## GRAFANA_SERVICE_PORT

```yaml
GRAFANA_SERVICE_PORT=80
```

Grafana web port.

Flow:

```text
Browser
    ↓
FrontendProxy
    ↓
Grafana
```

---

## IMAGE_PROVIDER_HOST

```yaml
IMAGE_PROVIDER_HOST=opentelemetry-demo-imageprovider
```

Image service hostname.

---

## IMAGE_PROVIDER_PORT

```yaml
IMAGE_PROVIDER_PORT=8081
```

Image provider port.

Used for:

```text
Product Images
Banner Images
Store Assets
```

---

## JAEGER_SERVICE_HOST

```yaml
JAEGER_SERVICE_HOST=opentelemetry-demo-jaeger-query
```

Jaeger UI hostname.

---

## JAEGER_SERVICE_PORT

```yaml
JAEGER_SERVICE_PORT=16686
```

Jaeger UI port.

Used for trace visualization.

Flow:

```text
Browser
    ↓
FrontendProxy
    ↓
Jaeger
```

---

## LOCUST_WEB_HOST

```yaml
LOCUST_WEB_HOST=opentelemetry-demo-loadgenerator
```

Load Generator hostname.

---

## LOCUST_WEB_PORT

```yaml
LOCUST_WEB_PORT=8089
```

Load Generator UI port.

Used for traffic simulation.

---

## OTEL_COLLECTOR_HOST

```yaml
OTEL_COLLECTOR_HOST=$(OTEL_COLLECTOR_NAME)
```

Resolves to:

```text
opentelemetry-demo-otelcol
```

---

## OTEL_COLLECTOR_PORT_GRPC

```yaml
OTEL_COLLECTOR_PORT_GRPC=4317
```

OTLP gRPC endpoint.

---

## OTEL_COLLECTOR_PORT_HTTP

```yaml
OTEL_COLLECTOR_PORT_HTTP=4318
```

OTLP HTTP endpoint.

---

## OTEL_RESOURCE_ATTRIBUTES

```yaml
service.name=$(OTEL_SERVICE_NAME)
service.namespace=opentelemetry-demo
service.version=1.12.0
```

Telemetry metadata.

---

# Resources

```yaml
resources:
  limits:
    memory: 50Mi
```

Maximum memory:

```text
50 MB
```

Reason:

FrontendProxy mainly routes requests.

It performs less business logic than Frontend.

---

# Security Context

```yaml
securityContext:
```

Security hardening.

---

## runAsGroup

```yaml
runAsGroup: 101
```

Linux group ID.

---

## runAsNonRoot

```yaml
runAsNonRoot: true
```

Container cannot run as root.

Security best practice.

---

## runAsUser

```yaml
runAsUser: 101
```

Runs as user ID 101.

---

# Volume Mounts

```yaml
volumeMounts:
```

No volumes attached.

---

# Volumes

```yaml
volumes:
```

No custom volumes.

---

# Complete Traffic Flow

```text
                    User Browser
                          │
                          ▼
                 FrontendProxy (8080)
                          │
         ┌────────────────┼─────────────────┐
         │                │                 │
         ▼                ▼                 ▼
     Frontend         Grafana           Jaeger
         │
         ▼
 ┌───────┼───────────────┬───────────────┐
 │       │               │               │
 ▼       ▼               ▼               ▼
Ad     Cart         Checkout      ProductCatalog
Svc    Svc           Service         Service

                          │
                          ▼
                     Flagd UI

                          │
                          ▼
                    Image Provider

                          │
                          ▼
                    Load Generator
```

# ImageProvider Deployment Documentation

## Purpose of ImageProvider Service

ImageProvider is responsible for serving images used by the OpenTelemetry Demo store.

Examples:

* Product images
* Advertisement images
* Store graphics
* Static media assets

Without ImageProvider:

```text
Products would load
But images would be missing
```

Users would see broken image placeholders.

---

# Metadata Section

```yaml
metadata:
  name: opentelemetry-demo-imageprovider
```

Deployment name.

---

# Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-imageprovider
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: imageprovider
  app.kubernetes.io/name: opentelemetry-demo-imageprovider
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

## Label Details

### opentelemetry.io/name

```yaml
opentelemetry.io/name: opentelemetry-demo-imageprovider
```

Unique identifier for ImageProvider.

---

### app.kubernetes.io/instance

```yaml
app.kubernetes.io/instance: opentelemetry-demo
```

Application instance.

---

### app.kubernetes.io/component

```yaml
app.kubernetes.io/component: imageprovider
```

Component name.

Used by:

```yaml
OTEL_SERVICE_NAME
```

Value becomes:

```text
imageprovider
```

---

### app.kubernetes.io/name

```yaml
app.kubernetes.io/name: opentelemetry-demo-imageprovider
```

Human readable name.

---

### app.kubernetes.io/version

```yaml
app.kubernetes.io/version: "1.12.0"
```

Application version.

---

### app.kubernetes.io/part-of

```yaml
app.kubernetes.io/part-of: opentelemetry-demo
```

Indicates membership in OpenTelemetry Demo application.

---

# Deployment Spec

## Replicas

```yaml
replicas: 1
```

Runs one pod.

---

## Revision History

```yaml
revisionHistoryLimit: 10
```

Keeps 10 deployment versions.

---

# Selector

```yaml
selector:
  matchLabels:
    opentelemetry.io/name: opentelemetry-demo-imageprovider
```

Deployment manages pods with this label.

---

# Pod Labels

```yaml
template:
  metadata:
    labels:
```

Same labels are attached to pod.

---

# Service Account

```yaml
serviceAccountName: opentelemetry-demo
```

Uses shared service account.

---

# Container Section

## Container Name

```yaml
name: imageprovider
```

Container identifier.

---

## Docker Image

```yaml
image: ghcr.io/open-telemetry/demo:1.12.0-imageprovider
```

Contains ImageProvider application.

---

## Image Pull Policy

```yaml
imagePullPolicy: IfNotPresent
```

Downloads image only if missing.

---

# Port Configuration

```yaml
ports:
  - containerPort: 8081
    name: service
```

Application listens on:

```text
8081
```

Flow:

```text
FrontendProxy
      ↓
ImageProvider
      ↓
Port 8081
```

---

# Environment Variables

## OTEL_SERVICE_NAME

Automatically derived from label:

```text
imageprovider
```

---

## OTEL_COLLECTOR_NAME

```yaml
OTEL_COLLECTOR_NAME=opentelemetry-demo-otelcol
```

Collector hostname.

---

## OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE

```yaml
cumulative
```

Metrics aggregation style.

---

## IMAGE_PROVIDER_PORT

```yaml
IMAGE_PROVIDER_PORT=8081
```

Listening port.

---

## OTEL_COLLECTOR_PORT_GRPC

```yaml
OTEL_COLLECTOR_PORT_GRPC=4317
```

OTLP gRPC port.

---

## OTEL_COLLECTOR_HOST

```yaml
OTEL_COLLECTOR_HOST=$(OTEL_COLLECTOR_NAME)
```

Resolves to:

```text
opentelemetry-demo-otelcol
```

---

## OTEL_RESOURCE_ATTRIBUTES

```yaml
service.name=imageprovider
service.namespace=opentelemetry-demo
service.version=1.12.0
```

Adds metadata to telemetry.

---

# Resource Limits

```yaml
resources:
  limits:
    memory: 50Mi
```

Maximum memory:

```text
50 MB
```

Reason:

* Only serves image files
* Very lightweight workload

---

# Volume Mounts

```yaml
volumeMounts:
```

None configured.

---

# Volumes

```yaml
volumes:
```

No custom volumes.

---

# ImageProvider Flow

```text
User Browser
      ↓
Frontend
      ↓
FrontendProxy
      ↓
ImageProvider
      ↓
Product Images
```

#  Kafka Deployment Documentation

## Purpose of Kafka Service

Kafka is the messaging backbone of the entire application.

Instead of services talking directly:

```text
Service A
   ↓
Service B
```

Kafka enables:

```text
Service A
   ↓
 Kafka
   ↓
Service B
```

This is called asynchronous communication.

---

# Why Kafka Is Needed

Several services depend on Kafka:

```text
Checkout Service
Accounting Service
Fraud Detection Service
```

Example Order Flow:

```text
Customer Places Order
         ↓
Checkout Service
         ↓
Publish Event To Kafka
         ↓
Accounting Service
         ↓
Create Accounting Record

Fraud Detection Service
         ↓
Analyze Order
```

Both services receive the same event independently.

---

# Metadata

```yaml
metadata:
  name: opentelemetry-demo-kafka
```

Deployment name.

---

# Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-kafka
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: kafka
  app.kubernetes.io/name: opentelemetry-demo-kafka
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

---

## Label Meaning

### Component

```yaml
app.kubernetes.io/component: kafka
```

Identifies component as Kafka broker.

---

### Version

```yaml
app.kubernetes.io/version: "1.12.0"
```

Kafka image version used in demo.

---

# Deployment Spec

## Replicas

```yaml
replicas: 1
```

Single Kafka broker.

Production environments usually run:

```text
3
5
7
```

brokers.

---

## Revision History

```yaml
revisionHistoryLimit: 10
```

Stores rollback history.

---

# Selector

```yaml
selector:
  matchLabels:
    opentelemetry.io/name: opentelemetry-demo-kafka
```

Matches Kafka pods.

---

# Service Account

```yaml
serviceAccountName: opentelemetry-demo
```

Uses common service account.

---

# Container

## Name

```yaml
name: kafka
```

Kafka broker container.

---

## Docker Image

```yaml
image: ghcr.io/open-telemetry/demo:1.12.0-kafka
```

Contains Kafka configured for demo environment.

---

## Image Pull Policy

```yaml
imagePullPolicy: IfNotPresent
```

Pull only if image absent.

---

# Container Ports

## Client Port

```yaml
containerPort: 9092
name: plaintext
```

Used by applications.

Services connect using:

```text
opentelemetry-demo-kafka:9092
```

Examples:

* Checkout Service
* Accounting Service
* Fraud Detection Service

---

## Controller Port

```yaml
containerPort: 9093
name: controller
```

Used internally by Kafka.

Responsible for:

* Metadata management
* Broker coordination
* Cluster control

---

# Environment Variables

## OTEL_SERVICE_NAME

Resolved from labels:

```text
kafka
```

---

## OTEL_COLLECTOR_NAME

```yaml
opentelemetry-demo-otelcol
```

Telemetry collector.

---

## KAFKA_ADVERTISED_LISTENERS

```yaml
KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://opentelemetry-demo-kafka:9092
```

Very important setting.

Tells clients:

```text
Connect using:

opentelemetry-demo-kafka:9092
```

Without this:

Clients may fail to connect.

---

## OTEL_EXPORTER_OTLP_ENDPOINT

```yaml
http://opentelemetry-demo-otelcol:4318
```

Exports telemetry.

---

## KAFKA_HEAP_OPTS

```yaml
-Xmx400M -Xms400M
```

JVM memory configuration.

### Xms

```text
400 MB
```

Initial heap size.

---

### Xmx

```text
400 MB
```

Maximum heap size.

So Kafka starts and stays at:

```text
400 MB
```

---

## OTEL_RESOURCE_ATTRIBUTES

```yaml
service.name=kafka
service.namespace=opentelemetry-demo
service.version=1.12.0
```

Telemetry metadata.

---

# Resource Limits

```yaml
resources:
  limits:
    memory: 600Mi
```

Kafka can consume:

```text
600 MB RAM
```

Largest memory allocation among many services.

Reason:

* Stores events
* Handles queues
* Manages message delivery
* Runs JVM

---

# Security Context

## runAsGroup

```yaml
runAsGroup: 1000
```

Linux group ID.

---

## runAsNonRoot

```yaml
runAsNonRoot: true
```

Cannot run as root.

---

## runAsUser

```yaml
runAsUser: 1000
```

Runs as user ID 1000.

---

# Volume Mounts

```yaml
volumeMounts:
```

No persistent storage defined in this deployment.

Important observation:

This Kafka is intended for:

```text
Demo Environment
Learning
Testing
```

Not production.

---

# Volumes

```yaml
volumes:
```

None configured.

---

# Kafka Communication Architecture

```text
                 Checkout Service
                        │
                        ▼
                    Kafka
                  Port 9092
                 /       \
                /         \
               ▼           ▼

   Accounting Service   Fraud Detection Service
```

# Recommendation Service Deployment Documentation

## Purpose of Recommendation Service

The Recommendation Service is responsible for generating product recommendations for users.

In an e-commerce application, when a customer views a product, this service suggests similar or related products that the customer may be interested in purchasing.

Examples:

* "Customers who bought this also bought..."
* "Recommended products"
* "You may also like"

This service communicates with:

* Product Catalog Service (to get product information)
* Flagd (for feature flags and experiments)
* OpenTelemetry Collector (for observability data)

---

# Complete YAML

```yaml
apiVersion: apps/v1
kind: Deployment
```

## apiVersion: apps/v1

Specifies that this resource uses the Kubernetes Deployment API.

`apps/v1` is the stable API version used for Deployments.

---

## kind: Deployment

Creates and manages Pods.

Deployment responsibilities:

* Creates Pods
* Maintains desired number of Pods
* Performs rolling updates
* Recovers failed Pods automatically

---

# Metadata Section

```yaml
metadata:
  name: opentelemetry-demo-recommendationservice
```

## name

Deployment name.

```text
opentelemetry-demo-recommendationservice
```

Kubernetes uses this name to identify the deployment.

---

# Labels Section

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-recommendationservice
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: recommendationservice
  app.kubernetes.io/name: opentelemetry-demo-recommendationservice
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

## Label Explanation

### opentelemetry.io/name

```yaml
opentelemetry.io/name: opentelemetry-demo-recommendationservice
```

Unique identifier for this service.

Used by:

* Services
* Monitoring systems
* Pod selectors

---

### app.kubernetes.io/instance

```yaml
app.kubernetes.io/instance: opentelemetry-demo
```

Indicates this service belongs to the OpenTelemetry Demo deployment.

---

### app.kubernetes.io/component

```yaml
app.kubernetes.io/component: recommendationservice
```

Identifies the business function.

Here:

```text
recommendationservice
```

---

### app.kubernetes.io/name

```yaml
app.kubernetes.io/name: opentelemetry-demo-recommendationservice
```

Human-readable application name.

---

### app.kubernetes.io/version

```yaml
app.kubernetes.io/version: "1.12.0"
```

Application version being deployed.

---

### app.kubernetes.io/part-of

```yaml
app.kubernetes.io/part-of: opentelemetry-demo
```

Indicates this deployment belongs to the larger OpenTelemetry Demo application.

---

# Deployment Specification

```yaml
spec:
```

Defines how the deployment should run.

---

## replicas

```yaml
replicas: 1
```

Only one Recommendation Service Pod will run.

Current state:

```text
Recommendation Service Pod = 1
```

---

## revisionHistoryLimit

```yaml
revisionHistoryLimit: 10
```

Stores up to 10 previous deployment revisions.

Benefits:

* Rollback support
* Deployment history tracking

---

# Selector Section

```yaml
selector:
  matchLabels:
    opentelemetry.io/name: opentelemetry-demo-recommendationservice
```

Deployment manages only Pods having this label.

Must match Pod template labels exactly.

---

# Pod Template

```yaml
template:
```

Template used to create Pods.

---

## Pod Labels

```yaml
metadata:
  labels:
```

Applied to all Pods created by this Deployment.

```yaml
opentelemetry.io/name: opentelemetry-demo-recommendationservice
app.kubernetes.io/instance: opentelemetry-demo
app.kubernetes.io/component: recommendationservice
app.kubernetes.io/name: opentelemetry-demo-recommendationservice
```

These labels are used by:

* Services
* Monitoring tools
* Selectors

---

# Pod Specification

```yaml
spec:
```

Defines what runs inside the Pod.

---

## Service Account

```yaml
serviceAccountName: opentelemetry-demo
```

Associates the Pod with the ServiceAccount:

```text
opentelemetry-demo
```

Provides Kubernetes identity and permissions.

---

# Container Section

```yaml
containers:
```

Defines the application container.

---

## Container Name

```yaml
name: recommendationservice
```

Container name inside the Pod.

---

## Container Image

```yaml
image: ghcr.io/open-telemetry/demo:1.12.0-recommendationservice
```

Image details:

| Part                         | Meaning                   |
| ---------------------------- | ------------------------- |
| ghcr.io                      | GitHub Container Registry |
| open-telemetry/demo          | Repository                |
| 1.12.0-recommendationservice | Service-specific image    |

This image contains Recommendation Service code.

---

## Image Pull Policy

```yaml
imagePullPolicy: IfNotPresent
```

Behavior:

* Use local image if available
* Download only if missing

Reduces startup time.

---

# Container Port

```yaml
ports:
  - containerPort: 8080
    name: service
```

Application listens on:

```text
Port 8080
```

Used by Kubernetes Service to route traffic.

---

# Environment Variables

## OTEL_SERVICE_NAME

```yaml
- name: OTEL_SERVICE_NAME
```

Obtained automatically from Pod labels.

Value becomes:

```text
recommendationservice
```

Used in traces and metrics.

---

## OTEL_COLLECTOR_NAME

```yaml
- name: OTEL_COLLECTOR_NAME
  value: opentelemetry-demo-otelcol
```

Collector service name.

Telemetry is sent here.

---

## OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE

```yaml
value: cumulative
```

Metrics are reported cumulatively over time.

---

## RECOMMENDATION_SERVICE_PORT

```yaml
value: "8080"
```

Application port configuration.

---

## PRODUCT_CATALOG_SERVICE_ADDR

```yaml
value: opentelemetry-demo-productcatalogservice:8080
```

Dependency on Product Catalog Service.

Purpose:

* Retrieve product information
* Generate recommendations

Communication Flow:

```text
Recommendation Service
        │
        ▼
Product Catalog Service
```

---

## OTEL_PYTHON_LOG_CORRELATION

```yaml
value: "true"
```

Enables log correlation.

Benefits:

* Links logs to traces
* Easier troubleshooting

Example:

```text
Trace
  └── Logs
```

---

## PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION

```yaml
value: python
```

Forces Python implementation of Protocol Buffers.

Often used for compatibility reasons.

---

## FLAGD_HOST

```yaml
value: opentelemetry-demo-flagd
```

Feature flag server hostname.

---

## FLAGD_PORT

```yaml
value: "8013"
```

Feature flag server port.

---

## OTEL_EXPORTER_OTLP_ENDPOINT

```yaml
value: http://$(OTEL_COLLECTOR_NAME):4317
```

Expands to:

```text
http://opentelemetry-demo-otelcol:4317
```

Telemetry destination.

---

## OTEL_RESOURCE_ATTRIBUTES

```yaml
service.name=$(OTEL_SERVICE_NAME),
service.namespace=opentelemetry-demo,
service.version=1.12.0
```

Metadata attached to telemetry.

Produces:

```text
service.name=recommendationservice
service.namespace=opentelemetry-demo
service.version=1.12.0
```

---

# Resource Limits

```yaml
resources:
  limits:
    memory: 500Mi
```

Maximum memory allowed:

```text
500 MiB
```

Why higher than many other services?

Because recommendation algorithms:

* Process product data
* Build recommendation lists
* Perform more computations

---

# Volume Mounts

```yaml
volumeMounts:
```

No volumes configured.

Container uses only internal filesystem.

---

# Volumes

```yaml
volumes:
```

No external storage attached.

---

# Overall Flow

```text
Customer Opens Product
           │
           ▼
Frontend
           │
           ▼
Recommendation Service
           │
           ▼
Product Catalog Service
           │
           ▼
Recommended Products Returned
           │
           ▼
Frontend Displays Suggestions
```

# Shipping Service Deployment Documentation

## Purpose of Shipping Service

The Shipping Service is responsible for calculating shipping costs and handling shipping-related operations during the checkout process.

When a customer places an order:

1. Checkout Service contacts Shipping Service.
2. Shipping Service calculates shipping charges.
3. Shipping Service may request shipping quotes from Quote Service.
4. Shipping details are returned to Checkout Service.

Without this service, the application would not be able to estimate or process delivery costs.

---

# Resource Information

```yaml
apiVersion: apps/v1
kind: Deployment
```

## apiVersion

```yaml
apiVersion: apps/v1
```

Uses Kubernetes Deployment API.

---

## kind

```yaml
kind: Deployment
```

Creates and manages Shipping Service Pods.

Responsibilities:

* Create Pods
* Maintain desired replicas
* Restart failed Pods
* Support rolling updates

---

# Metadata Section

```yaml
metadata:
  name: opentelemetry-demo-shippingservice
```

Deployment name:

```text
opentelemetry-demo-shippingservice
```

---

# Labels

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-shippingservice
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: shippingservice
  app.kubernetes.io/name: opentelemetry-demo-shippingservice
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

## Label Details

### opentelemetry.io/name

```yaml
opentelemetry.io/name: opentelemetry-demo-shippingservice
```

Unique identifier used by:

* Services
* Selectors
* Monitoring systems

---

### app.kubernetes.io/instance

```yaml
app.kubernetes.io/instance: opentelemetry-demo
```

Indicates this Deployment belongs to the OpenTelemetry Demo application.

---

### app.kubernetes.io/component

```yaml
app.kubernetes.io/component: shippingservice
```

Defines the business component:

```text
shippingservice
```

---

### app.kubernetes.io/name

```yaml
app.kubernetes.io/name: opentelemetry-demo-shippingservice
```

Human-readable application name.

---

### app.kubernetes.io/version

```yaml
app.kubernetes.io/version: "1.12.0"
```

Current application version.

---

### app.kubernetes.io/part-of

```yaml
app.kubernetes.io/part-of: opentelemetry-demo
```

Specifies that this service is part of the larger OpenTelemetry Demo application.

---

# Deployment Specification

## Replicas

```yaml
replicas: 1
```

Only one Shipping Service Pod is created.

```text
Desired Pods = 1
```

---

## Revision History

```yaml
revisionHistoryLimit: 10
```

Kubernetes keeps 10 previous Deployment versions.

Benefits:

* Rollback support
* Deployment history

---

# Selector

```yaml
selector:
  matchLabels:
    opentelemetry.io/name: opentelemetry-demo-shippingservice
```

Deployment manages only Pods containing this label.

Important:

Selector and Pod labels must match.

---

# Pod Template

## Pod Labels

```yaml
template:
  metadata:
    labels:
```

Applied to every Pod created by this Deployment.

```yaml
opentelemetry.io/name: opentelemetry-demo-shippingservice
app.kubernetes.io/instance: opentelemetry-demo
app.kubernetes.io/component: shippingservice
app.kubernetes.io/name: opentelemetry-demo-shippingservice
```

Used by:

* Kubernetes Service
* Monitoring systems
* Deployment selector

---

# Pod Specification

## Service Account

```yaml
serviceAccountName: opentelemetry-demo
```

Pod runs using:

```text
opentelemetry-demo
```

ServiceAccount.

Provides Kubernetes identity and permissions.

---

# Container Section

## Container Name

```yaml
name: shippingservice
```

Main application container.

---

## Container Image

```yaml
image: ghcr.io/open-telemetry/demo:1.12.0-shippingservice
```

### Breakdown

| Part                   | Meaning                   |
| ---------------------- | ------------------------- |
| ghcr.io                | GitHub Container Registry |
| open-telemetry/demo    | Repository                |
| 1.12.0-shippingservice | Shipping Service image    |

Contains Shipping Service application code.

---

## Image Pull Policy

```yaml
imagePullPolicy: IfNotPresent
```

Behavior:

* Use local image if available.
* Pull image only if missing.

---

# Container Port

```yaml
ports:
  - containerPort: 8080
    name: service
```

Shipping Service listens on:

```text
8080
```

Other services communicate using this port.

---

# Environment Variables

## OTEL_SERVICE_NAME

```yaml
valueFrom:
  fieldRef:
```

Automatically retrieves value from Pod label:

```text
shippingservice
```

Used for telemetry identification.

---

## OTEL_COLLECTOR_NAME

```yaml
value: opentelemetry-demo-otelcol
```

OpenTelemetry Collector hostname.

Telemetry is sent here.

---

## OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE

```yaml
value: cumulative
```

Metrics are reported cumulatively.

---

## SHIPPING_SERVICE_PORT

```yaml
value: "8080"
```

Application listening port.

---

## QUOTE_SERVICE_ADDR

```yaml
value: http://opentelemetry-demo-quoteservice:8080
```

Dependency on Quote Service.

### Purpose

Shipping Service contacts Quote Service to obtain shipping quotations.

Communication Flow:

```text
Shipping Service
       │
       ▼
Quote Service
       │
       ▼
Shipping Cost Returned
```

---

## OTEL_EXPORTER_OTLP_ENDPOINT

```yaml
value: http://$(OTEL_COLLECTOR_NAME):4317
```

Expands to:

```text
http://opentelemetry-demo-otelcol:4317
```

Telemetry destination.

Uses OTLP gRPC protocol.

---

## OTEL_RESOURCE_ATTRIBUTES

```yaml
service.name=$(OTEL_SERVICE_NAME),
service.namespace=opentelemetry-demo,
service.version=1.12.0
```

Metadata attached to traces, metrics, and logs.

Generated values:

```text
service.name=shippingservice
service.namespace=opentelemetry-demo
service.version=1.12.0
```

---

# Resource Limits

```yaml
resources:
  limits:
    memory: 20Mi
```

Maximum memory allowed:

```text
20 MiB
```

Reason:

Shipping Service performs lightweight operations:

* Receive requests
* Request shipping quotes
* Return shipping cost

Minimal memory requirements.

---

# Volume Mounts

```yaml
volumeMounts:
```

No volume mounts defined.

Uses container filesystem only.

---

# Volumes

```yaml
volumes:
```

No persistent or shared storage attached.

---

# Service Dependencies

This service directly depends on:

| Dependency              | Purpose                    |
| ----------------------- | -------------------------- |
| Quote Service           | Calculate shipping charges |
| OpenTelemetry Collector | Telemetry export           |

---

# Order Processing Flow

```text
Customer Places Order
          │
          ▼
Checkout Service
          │
          ▼
Shipping Service
          │
          ▼
Quote Service
          │
          ▼
Shipping Cost Generated
          │
          ▼
Checkout Service
          │
          ▼
Customer Receives Final Price
```

# Valkey Deployment Documentation

## Purpose of Valkey Service

Valkey is an in-memory key-value database used by the application for very fast data storage and retrieval.

In the OpenTelemetry Demo application, Valkey is primarily used by the Cart Service to store shopping cart information.

Why use Valkey?

* Extremely fast (memory-based storage)
* Low latency
* Ideal for session and cart data
* Reduces load on backend services

Example:

```text
User Adds Product to Cart
           │
           ▼
Cart Service
           │
           ▼
Valkey Stores Cart Data
```

Without Valkey, the Cart Service would need a slower database for every cart operation.

---

# Resource Type

```yaml
apiVersion: apps/v1
kind: Deployment
```

## apiVersion

```yaml
apiVersion: apps/v1
```

Uses Kubernetes Deployment API.

---

## kind

```yaml
kind: Deployment
```

Creates and manages Valkey Pods.

Responsibilities:

* Create Pods
* Maintain desired replicas
* Restart failed Pods
* Perform rolling updates

---

# Metadata Section

## Deployment Name

```yaml
metadata:
  name: opentelemetry-demo-valkey
```

Deployment name:

```text
opentelemetry-demo-valkey
```

Used by Kubernetes to identify this Deployment.

---

# Labels Section

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo-valkey
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/component: valkey
  app.kubernetes.io/name: opentelemetry-demo-valkey
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

## Label Details

### opentelemetry.io/name

```yaml
opentelemetry.io/name: opentelemetry-demo-valkey
```

Unique identifier for the Valkey component.

Used by:

* Services
* Selectors
* Monitoring tools

---

### app.kubernetes.io/instance

```yaml
app.kubernetes.io/instance: opentelemetry-demo
```

Indicates that this Deployment belongs to the OpenTelemetry Demo application.

---

### app.kubernetes.io/component

```yaml
app.kubernetes.io/component: valkey
```

Specifies the component type.

Value:

```text
valkey
```

---

### app.kubernetes.io/name

```yaml
app.kubernetes.io/name: opentelemetry-demo-valkey
```

Human-readable application name.

---

### app.kubernetes.io/version

```yaml
app.kubernetes.io/version: "1.12.0"
```

Application version.

---

### app.kubernetes.io/part-of

```yaml
app.kubernetes.io/part-of: opentelemetry-demo
```

Indicates that Valkey is part of the OpenTelemetry Demo platform.

---

# Deployment Specification

## Replicas

```yaml
replicas: 1
```

One Valkey Pod will run.

```text
Desired Pods = 1
```

---

## Revision History

```yaml
revisionHistoryLimit: 10
```

Kubernetes stores 10 previous Deployment versions.

Benefits:

* Rollback capability
* Deployment history tracking

---

# Selector

```yaml
selector:
  matchLabels:
    opentelemetry.io/name: opentelemetry-demo-valkey
```

Deployment manages Pods having this label.

Important:

```text
Selector Labels
       =
Pod Labels
```

Otherwise Deployment cannot manage its Pods.

---

# Pod Template

## Pod Labels

```yaml
template:
  metadata:
    labels:
```

Applied to every Valkey Pod.

```yaml
opentelemetry.io/name: opentelemetry-demo-valkey
app.kubernetes.io/instance: opentelemetry-demo
app.kubernetes.io/component: valkey
app.kubernetes.io/name: opentelemetry-demo-valkey
```

Used by:

* Kubernetes Services
* Monitoring tools
* Deployment selectors

---

# Pod Specification

## Service Account

```yaml
serviceAccountName: opentelemetry-demo
```

Pod uses:

```text
opentelemetry-demo
```

ServiceAccount.

Provides Pod identity inside Kubernetes.

---

# Container Section

## Container Name

```yaml
name: valkey
```

Main Valkey database container.

---

## Container Image

```yaml
image: valkey/valkey:7.2-alpine
```

### Image Breakdown

| Part   | Meaning                 |
| ------ | ----------------------- |
| valkey | Repository              |
| 7.2    | Valkey Version          |
| alpine | Lightweight Linux image |

---

### Why Alpine?

Alpine Linux is:

* Small image size
* Fast downloads
* Lower memory usage
* Better security surface

---

## Image Pull Policy

```yaml
imagePullPolicy: IfNotPresent
```

Behavior:

* Use local image if already downloaded.
* Pull image only when missing.

---

# Container Port

```yaml
ports:
  - containerPort: 6379
    name: valkey
```

Valkey listens on:

```text
6379
```

This is the default Valkey/Redis port.

Cart Service connects using:

```text
opentelemetry-demo-valkey:6379
```

---

# Environment Variables

## OTEL_SERVICE_NAME

```yaml
valueFrom:
  fieldRef:
```

Reads value from Pod labels.

Result:

```text
valkey
```

Used for observability.

---

## OTEL_COLLECTOR_NAME

```yaml
value: opentelemetry-demo-otelcol
```

OpenTelemetry Collector hostname.

Telemetry data is sent here.

---

## OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE

```yaml
value: cumulative
```

Metrics are exported as cumulative values.

---

## OTEL_RESOURCE_ATTRIBUTES

```yaml
service.name=$(OTEL_SERVICE_NAME),
service.namespace=opentelemetry-demo,
service.version=1.12.0
```

Adds metadata to telemetry.

Generated values:

```text
service.name=valkey
service.namespace=opentelemetry-demo
service.version=1.12.0
```

---

# Resource Limits

```yaml
resources:
  limits:
    memory: 20Mi
```

Maximum memory allowed:

```text
20 MiB
```

Why so small?

Because this demo environment:

* Stores limited cart data
* Handles lightweight workloads
* Optimized for demonstrations

Production environments usually allocate much more memory.

---

# Security Context

```yaml
securityContext:
  runAsGroup: 1000
  runAsNonRoot: true
  runAsUser: 999
```

This improves container security.

---

## runAsNonRoot

```yaml
runAsNonRoot: true
```

Prevents the container from running as root.

Security benefit:

```text
Less privilege
=
Less risk
```

---

## runAsUser

```yaml
runAsUser: 999
```

Container runs as Linux user ID:

```text
999
```

instead of root.

---

## runAsGroup

```yaml
runAsGroup: 1000
```

Processes belong to group:

```text
1000
```

Used for file permissions.

---

# Volume Mounts

```yaml
volumeMounts:
```

No volume mounts are configured.

Valkey uses only container-local storage.

---

# Volumes

```yaml
volumes:
```

No persistent storage attached.

Important:

If Pod is deleted:

```text
Valkey Data
     ↓
Lost
```

This is acceptable because:

* Demo environment
* Cart data is temporary
* Persistence is not required

---

# Communication Flow

```text
User Adds Item To Cart
          │
          ▼
Cart Service
          │
          ▼
Valkey
(Store Cart Data)
          │
          ▼
Cart Retrieved Later
```

---

# Dependency Relationship

```text
Cart Service
      │
      ▼
Valkey
```

From the Cart Service deployment:

```yaml
VALKEY_ADDR=opentelemetry-demo-valkey:6379
```

This is how Cart Service connects to Valkey.

---

# ConfigMap Documentation – opentelemetry-demo-flagd-config

This is the final configuration file used by the **flagd feature flag service**. Instead of creating Pods or Services, this YAML stores configuration data that the `flagd` Deployment reads at runtime.

---

# YAML Structure

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
data:
```

| Field           | Meaning                         |
| --------------- | ------------------------------- |
| apiVersion: v1  | Uses Kubernetes core API        |
| kind: ConfigMap | Stores configuration data       |
| metadata        | Information about the ConfigMap |
| data            | Actual configuration content    |

---

# Metadata Section

```yaml
metadata:
  name: opentelemetry-demo-flagd-config
```

ConfigMap name.

Used by the Flagd deployment:

```yaml
volumes:
  - configMap:
      name: opentelemetry-demo-flagd-config
```

This mounts the ConfigMap into the container filesystem.

---

# Labels Section

```yaml
labels:
  opentelemetry.io/name: opentelemetry-demo
  app.kubernetes.io/instance: opentelemetry-demo
  app.kubernetes.io/name: opentelemetry-demo
  app.kubernetes.io/version: "1.12.0"
  app.kubernetes.io/part-of: opentelemetry-demo
```

### Purpose

Used for:

* Resource identification
* Monitoring
* Searching resources
* Grouping resources

### Label Details

| Label                      | Purpose               |
| -------------------------- | --------------------- |
| opentelemetry.io/name      | Demo application name |
| app.kubernetes.io/instance | Deployment instance   |
| app.kubernetes.io/name     | Application name      |
| app.kubernetes.io/version  | Application version   |
| app.kubernetes.io/part-of  | Parent application    |

---

# Data Section

```yaml
data:
  demo.flagd.json: |
```

Creates a file named:

```text
demo.flagd.json
```

inside the container.

Flagd reads this file to determine which feature flags exist.

---

# JSON Schema

```json
"$schema": "https://flagd.dev/schema/v0/flags.json"
```

Tells flagd that the JSON follows the official Flagd feature flag format.

---

# Flags Section

```json
"flags": {
}
```

Contains all feature flags.

A feature flag is a runtime switch that can enable/disable functionality without redeploying applications.

---

# Common Flag Structure

Most flags follow:

```json
{
  "description": "...",
  "state": "ENABLED",
  "variants": {
      "on": true,
      "off": false
  },
  "defaultVariant": "off"
}
```

### Meaning

| Field          | Purpose                      |
| -------------- | ---------------------------- |
| description    | Explains flag purpose        |
| state          | Flag is active and available |
| variants       | Possible values              |
| defaultVariant | Value used by default        |

---

# productCatalogFailure

```json
"productCatalogFailure"
```

### Purpose

Simulates Product Catalog service failure.

When enabled:

```json
"on": true
```

Product catalog requests intentionally fail.

Default:

```json
"off"
```

Service works normally.

---

# recommendationServiceCacheFailure

```json
"recommendationServiceCacheFailure"
```

### Purpose

Simulates cache failures inside Recommendation Service.

Useful for:

* Testing resilience
* Observing traces
* Monitoring cache errors

Default:

```json
off
```

---

# adServiceManualGc

```json
"adServiceManualGc"
```

### Purpose

Forces manual garbage collection in Ad Service.

Effect:

* Increased GC activity
* Performance degradation
* Observable telemetry events

Used for performance testing.

---

# adServiceHighCpu

```json
"adServiceHighCpu"
```

### Purpose

Artificially increases CPU usage in Ad Service.

When enabled:

```json
on = true
```

Ad Service consumes high CPU resources.

Useful for:

* Load testing
* Alert testing
* CPU monitoring demonstrations

---

# adServiceFailure

```json
"adServiceFailure"
```

### Purpose

Forces Ad Service to fail requests.

Useful for:

* Failure testing
* Distributed tracing demonstrations
* Error monitoring validation

---

# kafkaQueueProblems

```json
"kafkaQueueProblems"
```

### Purpose

Creates Kafka backlog problems.

Variants:

```json
"on": 100
"off": 0
```

When enabled:

* Produces excessive Kafka messages
* Introduces consumer delay
* Creates queue lag

Useful for:

* Kafka monitoring
* Consumer lag testing
* Alert testing

Default:

```json
off
```

---

# cartServiceFailure

```json
"cartServiceFailure"
```

### Purpose

Simulates Shopping Cart service failure.

When enabled:

* Add-to-cart may fail
* Cart retrieval may fail

Default:

```json
off
```

---

# paymentServiceFailure

```json
"paymentServiceFailure"
```

### Purpose

Payment requests fail intentionally.

Examples:

* Payment declined
* Payment processing errors

Useful for testing checkout failures.

Default:

```json
off
```

---

# paymentServiceUnreachable

```json
"paymentServiceUnreachable"
```

### Purpose

Makes Payment Service unavailable.

Difference from previous flag:

| Flag                      | Behavior                      |
| ------------------------- | ----------------------------- |
| paymentServiceFailure     | Service responds with failure |
| paymentServiceUnreachable | Service cannot be reached     |

This simulates:

* Network issues
* Service crash
* Service downtime

---

# loadgeneratorFloodHomepage

```json
"loadgeneratorFloodHomepage"
```

### Purpose

Creates heavy traffic against frontend.

Variants:

```json
on = 100
off = 0
```

When enabled:

* Load Generator floods homepage
* Request rate increases dramatically

Useful for:

* Stress testing
* Performance testing
* Observability demonstrations

---

# imageSlowLoad

```json
"imageSlowLoad"
```

### Purpose

Artificially slows image loading.

Variants:

```json
"10sec": 10000
"5sec": 5000
"off": 0
```

### Meaning

| Variant | Delay     |
| ------- | --------- |
| 10sec   | 10,000 ms |
| 5sec    | 5,000 ms  |
| off     | No delay  |

Example:

```json
"defaultVariant": "off"
```

Images load normally unless changed.

Useful for:

* Frontend performance testing
* User experience testing
* Distributed tracing demonstrations

---

# How This ConfigMap Is Used

Flow:

```text
ConfigMap
    ↓
Mounted into Flagd Pod
    ↓
Creates demo.flagd.json
    ↓
Flagd reads flags
    ↓
Microservices query Flagd
    ↓
Flagd returns ON/OFF values
    ↓
Services change behavior dynamically
```

Example:

```text
Cart Service
    ↓
asks Flagd:
"cartServiceFailure" ?
    ↓
Flagd returns:
off
    ↓
Normal operation
```

or

```text
Cart Service
    ↓
asks Flagd:
"cartServiceFailure" ?
    ↓
Flagd returns:
on
    ↓
Intentional failure generated
```


