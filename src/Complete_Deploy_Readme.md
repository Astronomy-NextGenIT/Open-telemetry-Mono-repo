Introduction to the Deployment File
What is this file?

This file is a Kubernetes Deployment Manifest that contains all the Kubernetes resources required to deploy the OpenTelemetry Demo application into a Kubernetes cluster.

Instead of creating resources one by one, all resources are defined in a single YAML file and applied together using:

kubectl apply -f complete-deploy.yaml

When Kubernetes reads this file, it creates all required objects such as:

ServiceAccount
ConfigMap
Services
Deployments
Pods

These resources work together to run the complete microservices application.

Why do we need this file?

Imagine you have an e-commerce application consisting of multiple microservices:

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

Each microservice needs:

A container image
Network configuration
Environment variables
Kubernetes Service
Deployment configuration

Managing all these manually would be difficult.

Therefore Kubernetes uses YAML manifest files to describe the desired state of the application.

What happens when this file is applied?

When you run:

kubectl apply -f complete-deploy.yaml

Kubernetes performs the following actions:

Step 1: Creates ServiceAccount
ServiceAccount
     │
     ▼
Provides identity to Pods

Pods use this identity when communicating with the Kubernetes API.

Step 2: Creates ConfigMaps
ConfigMap
     │
     ▼
Stores application configuration

Instead of hardcoding configuration inside containers, Kubernetes stores it separately.

Step 3: Creates Deployments
Deployment
     │
     ▼
Creates Pods

Example:

Deployment
    │
    ▼
Pod
    │
    ▼
Container

Deployments ensure required Pods are always running.

Step 4: Creates Services
Service
     │
     ▼
Provides stable networking

Pods can die and get recreated.

Their IP addresses change.

Services provide a permanent name such as:

cartservice
paymentservice
frontend

so applications can communicate reliably.
