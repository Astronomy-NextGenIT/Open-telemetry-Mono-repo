# Cart Service CI/CD Pipeline Using Amazon ECR Public and Kubernetes

## Overview

This Jenkins pipeline automates the complete build, containerization, image publishing, and deployment process for the Cart Service application. The pipeline retrieves the source code from GitHub, builds a Docker image, publishes the image to Amazon ECR Public, updates the Kubernetes deployment manifest with the newly generated image tag, deploys the application to the Kubernetes cluster, and verifies successful rollout.

The pipeline ensures that every build generates a uniquely tagged container image, enabling version tracking, rollback capability, and consistent deployment across environments.

---

# Pipeline Configuration

## Environment Variables

The pipeline uses the following environment variables:

| Variable       | Description                                               |
| -------------- | --------------------------------------------------------- |
| AWS_REGION     | AWS region used for Amazon ECR Public authentication      |
| PUBLIC_ECR_URI | URI of the Amazon ECR Public repository                   |
| IMAGE_NAME     | Name of the Docker image being built                      |
| IMAGE_TAG      | Unique image tag generated using the Jenkins build number |
| K8S_DEPLOYMENT | Kubernetes deployment name for Cart Service               |
| NAMESPACE      | Kubernetes namespace where the application is deployed    |

### Configured Values

```text
AWS_REGION=us-east-1

PUBLIC_ECR_URI=public.ecr.aws/q2a1e2p7/open-telemetry-demo

IMAGE_NAME=cart

IMAGE_TAG=cart-${BUILD_NUMBER}

K8S_DEPLOYMENT=opentelemetry-demo-cartservice

NAMESPACE=default
```

---
# Complete pipeline

```groovy
pipeline {
    agent any

    environment {

        AWS_REGION = "us-east-1"

        PUBLIC_ECR_URI = "public.ecr.aws/q2a1e2p7/open-telemetry-demo"

        IMAGE_NAME = "cart"
        IMAGE_TAG = "${IMAGE_NAME}-${BUILD_NUMBER}"

        K8S_DEPLOYMENT = "opentelemetry-demo-cartservice"
        NAMESPACE = "default"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Astronomy-NextGenIT/Open-telemetry-Mono-repo.git'
            }
        }

        stage('Verify Tools') {
            steps {
                sh '''
                aws --version
                kubectl version --client
                docker --version
                '''
            }
        }

        stage('Login Public ECR') {
            steps {
                sh '''
                aws ecr-public get-login-password \
                --region ${AWS_REGION} | \
                docker login \
                --username AWS \
                --password-stdin public.ecr.aws
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build \
                -t ${IMAGE_NAME}:${IMAGE_TAG} \
                -f src/cart/src/Dockerfile .
                '''
            }
        }

        stage('Tag Image') {
            steps {
                sh '''
                docker tag \
                ${IMAGE_NAME}:${IMAGE_TAG} \
                ${PUBLIC_ECR_URI}:${IMAGE_TAG}

                docker tag \
                ${IMAGE_NAME}:${IMAGE_TAG} \
                ${PUBLIC_ECR_URI}:${IMAGE_NAME}-latest
                '''
            }
        }

        stage('Push Image To Public ECR') {
            steps {
                sh '''
                docker push ${PUBLIC_ECR_URI}:${IMAGE_TAG}
                docker push ${PUBLIC_ECR_URI}:${IMAGE_NAME}-latest
                '''
            }
        }

        stage('Update Manifest') {
            steps {
                sh '''
                sed -i "s|image:.*|image: ${PUBLIC_ECR_URI}:${IMAGE_TAG}|g" \
                src/cart/kubernetes/cart/deploy.yaml

                echo "Updated Image:"
                grep -n "image:" src/cart/kubernetes/cart/deploy.yaml
                '''
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                sh '''
                kubectl apply -f src/cart/kubernetes/cart/deploy.yaml
                kubectl apply -f src/cart/kubernetes/cart/svc.yaml
                '''
            }
        }

        stage('Rollout Status') {
            steps {
                sh '''
                kubectl rollout status deployment/${K8S_DEPLOYMENT} \
                -n ${NAMESPACE} \
                --timeout=300s
                '''
            }
        }
    }

    post {

        success {
            echo "Cart Service Deployment Successful"
            echo "Image Pushed: ${PUBLIC_ECR_URI}:${IMAGE_TAG}"
        }

        failure {
            echo "Pipeline Failed"
        }

        always {
            sh '''
            docker image prune -af || true
            '''
        }
    }
}

```
# Pipeline Workflow

The pipeline consists of multiple stages that execute sequentially.

## Stage 1: Checkout Source Code

### Purpose

Retrieves the latest source code from the GitHub repository.

### Actions Performed

* Connects to GitHub repository.
* Checks out the main branch.
* Downloads the latest application source code into the Jenkins workspace.

### Command Executed

```bash
git branch: 'main'
```

### Outcome

Latest Cart Service source code becomes available for build and deployment.

---

## Stage 2: Verify Required Tools

### Purpose

Ensures that all required tools are installed and accessible on the Jenkins agent.

### Tools Verified

* AWS CLI
* kubectl
* Docker

### Commands Executed

```bash
aws --version

kubectl version --client

docker --version
```

### Outcome

Validation that the Jenkins agent has all required dependencies before proceeding.

---

## Stage 3: Authenticate with Amazon ECR Public

### Purpose

Authenticates Docker with Amazon ECR Public so that images can be pushed successfully.

### Actions Performed

* Generates an authentication token using AWS CLI.
* Passes the token securely to Docker.
* Logs Docker into Amazon ECR Public Registry.

### Command Executed

```bash
aws ecr-public get-login-password \
--region us-east-1 | \
docker login \
--username AWS \
--password-stdin public.ecr.aws
```

### Outcome

Docker receives authorization to push images to the public repository.

---

## Stage 4: Build Docker Image

### Purpose

Builds the Cart Service container image.

### Actions Performed

* Uses the Cart Service Dockerfile.
* Compiles the application.
* Creates a Docker image with a unique build tag.

### Dockerfile Location

```text
src/cart/src/Dockerfile
```

### Build Command

```bash
docker build \
-t cart:cart-${BUILD_NUMBER} \
-f src/cart/src/Dockerfile .
```

### Outcome

A Docker image is generated locally on the Jenkins build agent.

---

## Stage 5: Tag Docker Image

### Purpose

Creates repository tags required for publishing images to Amazon ECR Public.

### Tags Generated

#### Build-Specific Tag

```text
public.ecr.aws/q2a1e2p7/open-telemetry-demo:cart-BUILD_NUMBER
```

#### Latest Tag

```text
public.ecr.aws/q2a1e2p7/open-telemetry-demo:cart-latest
```

### Commands Executed

```bash
docker tag \
cart:cart-${BUILD_NUMBER} \
public.ecr.aws/q2a1e2p7/open-telemetry-demo:cart-${BUILD_NUMBER}

docker tag \
cart:cart-${BUILD_NUMBER} \
public.ecr.aws/q2a1e2p7/open-telemetry-demo:cart-latest
```

### Outcome

The image becomes ready for publication to Amazon ECR Public.

---

## Stage 6: Push Image to Amazon ECR Public

### Purpose

Publishes the Docker image to the public container registry.

### Actions Performed

Pushes two image versions:

1. Build-specific image
2. Latest image

### Commands Executed

```bash
docker push public.ecr.aws/q2a1e2p7/open-telemetry-demo:cart-${BUILD_NUMBER}

docker push public.ecr.aws/q2a1e2p7/open-telemetry-demo:cart-latest
```

### Outcome

Container images become publicly available for deployment.

---

## Stage 7: Update Kubernetes Manifest

### Purpose

Updates the deployment manifest with the newly generated image tag.

### Actions Performed

* Locates the image field in the deployment YAML.
* Replaces the existing image reference.
* Inserts the newly pushed image tag.

### Manifest File

```text
src/cart/kubernetes/cart/deploy.yaml
```

### Command Executed

```bash
sed -i "s|image:.*|image: public.ecr.aws/q2a1e2p7/open-telemetry-demo:cart-${BUILD_NUMBER}|g" \
src/cart/kubernetes/cart/deploy.yaml
```

### Verification

```bash
grep -n "image:" src/cart/kubernetes/cart/deploy.yaml
```

### Outcome

Deployment manifest references the latest image generated by the pipeline.

---

## Stage 8: Deploy to Kubernetes

### Purpose

Applies Kubernetes resources to the cluster.

### Resources Deployed

#### Deployment

```text
src/cart/kubernetes/cart/deploy.yaml
```

#### Service

```text
src/cart/kubernetes/cart/svc.yaml
```

### Commands Executed

```bash
kubectl apply -f src/cart/kubernetes/cart/deploy.yaml

kubectl apply -f src/cart/kubernetes/cart/svc.yaml
```

### Outcome

Kubernetes updates the Cart Service deployment using the newly published container image.

---

## Stage 9: Verify Rollout Status

### Purpose

Confirms that the deployment has been successfully rolled out.

### Actions Performed

* Monitors deployment progress.
* Waits for updated pods to become healthy.
* Verifies successful deployment.

### Command Executed

```bash
kubectl rollout status \
deployment/opentelemetry-demo-cartservice \
-n default \
--timeout=300s
```

### Outcome

The pipeline confirms successful deployment or reports rollout failures.

---

# Post-Build Actions

## Success Handling

When all stages complete successfully:

```text
Cart Service Deployment Successful
Image Pushed: public.ecr.aws/q2a1e2p7/open-telemetry-demo:cart-BUILD_NUMBER
```

### Purpose

Provides deployment confirmation and image details.

---

## Failure Handling

If any stage fails:

```text
Pipeline Failed
```

### Purpose

Clearly indicates deployment failure for troubleshooting.

---

## Workspace Cleanup

### Purpose

Removes unused Docker images from the Jenkins agent to free disk space.

### Command Executed

```bash
docker image prune -af || true
```

### Outcome

Prevents disk space exhaustion on Jenkins build servers.

---

# Pipeline Benefits

## Automated Build Process

Eliminates manual image creation and deployment activities.

## Versioned Container Images

Each build receives a unique tag based on the Jenkins build number.

## Latest Image Tracking

Maintains a dedicated latest tag for quick access to the most recent version.

## Consistent Deployments

Ensures Kubernetes always deploys the exact image generated by the pipeline.

## Automated Rollout Validation

Verifies application availability after deployment.

## Reduced Operational Effort

Provides an end-to-end automated CI/CD workflow from source code checkout to production deployment.

---

# Deployment Flow Summary

```text
GitHub Repository
        │
        ▼
Checkout Source Code
        │
        ▼
Verify Tools
        │
        ▼
Authenticate with Amazon ECR Public
        │
        ▼
Build Docker Image
        │
        ▼
Tag Image
        │
        ▼
Push Image to Amazon ECR Public
        │
        ▼
Update Kubernetes Manifest
        │
        ▼
Deploy to Kubernetes
        │
        ▼
Verify Rollout Status
        │
        ▼
Application Successfully Running
```
