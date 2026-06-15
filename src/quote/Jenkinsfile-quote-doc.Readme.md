# OpenTelemetry Demo – Quoteservice CI/CD Pipeline Documentation

## Overview

This Jenkins Pipeline automates the complete CI/CD process for the Quoteservice microservice within the OpenTelemetry Demo application. The pipeline performs source code checkout, Docker image creation, image publishing to Amazon Public ECR, Kubernetes manifest updates, deployment to the Kubernetes cluster, and deployment verification.

---

# Pipeline Configuration

## Environment Variables

The pipeline uses the following environment variables:

| Variable       | Description                                                   |
| -------------- | ------------------------------------------------------------- |
| AWS_REGION     | AWS region used for Public ECR authentication                 |
| PUBLIC_ECR_URI | Public ECR repository URI                                     |
| IMAGE_NAME     | Docker image name for Quoteservice                            |
| IMAGE_TAG      | Build-specific image tag generated using Jenkins build number |
| K8S_DEPLOYMENT | Kubernetes deployment name                                    |
| NAMESPACE      | Kubernetes namespace                                          |

```groovy
environment {

    AWS_REGION = "us-east-1"

    PUBLIC_ECR_URI = "public.ecr.aws/q2a1e2p7/open-telemetry-demo"

    IMAGE_NAME = "quoteservice"
    IMAGE_TAG = "${IMAGE_NAME}-${BUILD_NUMBER}"

    K8S_DEPLOYMENT = "opentelemetry-demo-quoteservice"
    NAMESPACE = "default"
}
```

---

# Stage 1 – Checkout Source Code

## Purpose

Downloads the latest application source code from the GitHub repository.

## Jenkins Stage

```groovy
stage('Checkout') {
    steps {
        git branch: 'main',
            url: 'https://github.com/Astronomy-NextGenIT/Open-telemetry-Mono-repo.git'
    }
}
```

## Actions Performed

1. Connects to GitHub.
2. Checks out the main branch.
3. Downloads the latest version of the Quoteservice source code.
4. Makes source files available in the Jenkins workspace.

---

# Stage 2 – Verify Required Tools

## Purpose

Verifies that all required tools are installed and accessible on the Jenkins server.

## Jenkins Stage

```groovy
stage('Verify Tools') {
    steps {
        sh '''
        aws --version
        kubectl version --client
        docker --version
        '''
    }
}
```

## Actions Performed

1. Checks AWS CLI installation.
2. Checks Kubernetes CLI installation.
3. Checks Docker installation.
4. Stops the pipeline if any required tool is unavailable.

---

# Stage 3 – Login to Amazon Public ECR

## Purpose

Authenticates Docker with Amazon Public Elastic Container Registry.

## Jenkins Stage

```groovy
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
```

## Actions Performed

1. Generates ECR authentication token.
2. Passes token securely to Docker.
3. Establishes Docker authentication session.
4. Enables image push operations.

---

# Stage 4 – Build Docker Image

## Purpose

Builds the Quoteservice Docker image using the service Dockerfile.

## Jenkins Stage

```groovy
stage('Build Docker Image') {
    steps {
        sh '''
        docker build \
        -t ${IMAGE_NAME}:${IMAGE_TAG} \
        -f src/quote/Dockerfile .
        '''
    }
}
```

## Actions Performed

1. Reads Dockerfile from:

```text
src/quote/Dockerfile
```

2. Uses repository root as build context.

3. Creates Docker image:

```text
quoteservice:<build-number>
```

Example:

```text
quoteservice:quoteservice-15
```

---

# Stage 5 – Tag Docker Image

## Purpose

Creates ECR-compatible image tags.

## Jenkins Stage

```groovy
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
```

## Actions Performed

Creates two tags:

### Build-Specific Tag

```text
public.ecr.aws/q2a1e2p7/open-telemetry-demo:quoteservice-15
```

### Latest Tag

```text
public.ecr.aws/q2a1e2p7/open-telemetry-demo:quoteservice-latest
```

This allows version tracking and latest image deployments.

---

# Stage 6 – Push Docker Image to Public ECR

## Purpose

Publishes Docker images to Amazon Public ECR.

## Jenkins Stage

```groovy
stage('Push Image To Public ECR') {
    steps {
        sh '''
        docker push ${PUBLIC_ECR_URI}:${IMAGE_TAG}
        docker push ${PUBLIC_ECR_URI}:${IMAGE_NAME}-latest
        '''
    }
}
```

## Actions Performed

Pushes:

```text
public.ecr.aws/q2a1e2p7/open-telemetry-demo:quoteservice-<build-number>
```

and

```text
public.ecr.aws/q2a1e2p7/open-telemetry-demo:quoteservice-latest
```

to Amazon Public ECR.

---

# Stage 7 – Update Kubernetes Manifest

## Purpose

Updates the deployment manifest with the newly built Docker image.

## Jenkins Stage

```groovy
stage('Update Manifest') {
    steps {
        sh '''
        sed -i "s|image:.*|image: ${PUBLIC_ECR_URI}:${IMAGE_TAG}|g" \
        src/quote/kubernetes/quote/deploy.yaml

        echo "Updated Image:"
        grep -n "image:" \
        src/quote/kubernetes/quote/deploy.yaml
        '''
    }
}
```

## Actions Performed

1. Opens deployment manifest.

```text
src/quote/kubernetes/quote/deploy.yaml
```

2. Replaces existing image reference.

3. Injects newly generated image tag.

Example:

Before:

```yaml
image: ghcr.io/open-telemetry/demo:1.12.0-quoteservice
```

After:

```yaml
image: public.ecr.aws/q2a1e2p7/open-telemetry-demo:quoteservice-15
```

4. Displays updated image configuration.

---

# Stage 8 – Deploy to Kubernetes

## Purpose

Deploys Quoteservice resources to the Kubernetes cluster.

## Jenkins Stage

```groovy
stage('Deploy To Kubernetes') {
    steps {
        sh '''
        kubectl apply -f src/quote/kubernetes/quote/deploy.yaml
        kubectl apply -f src/quote/kubernetes/quote/svc.yaml
        '''
    }
}
```

## Actions Performed

Deploys:

### Deployment Resource

```text
src/quote/kubernetes/quote/deploy.yaml
```

### Service Resource

```text
src/quote/kubernetes/quote/svc.yaml
```

Updates Kubernetes cluster with the latest Quoteservice image.

---

# Stage 9 – Verify Rollout Status

## Purpose

Ensures deployment completes successfully.

## Jenkins Stage

```groovy
stage('Rollout Status') {
    steps {
        sh '''
        kubectl rollout status \
        deployment/${K8S_DEPLOYMENT} \
        -n ${NAMESPACE} \
        --timeout=300s
        '''
    }
}
```

## Actions Performed

1. Monitors deployment progress.
2. Waits for pod creation.
3. Waits for containers to become ready.
4. Fails the pipeline if deployment exceeds 300 seconds.

Deployment monitored:

```text
opentelemetry-demo-quoteservice
```

---

# Post Build Actions

## Success

```groovy
success {
    echo "Quoteservice Deployment Successful"
    echo "Image Pushed: ${PUBLIC_ECR_URI}:${IMAGE_TAG}"
}
```

Displays:

```text
Quoteservice Deployment Successful
Image Pushed: public.ecr.aws/q2a1e2p7/open-telemetry-demo:quoteservice-15
```

---

## Failure

```groovy
failure {
    echo "Quoteservice Pipeline Failed"
}
```

Displays failure notification if any stage fails.

---

## Cleanup

```groovy
always {
    sh '''
    docker image prune -af || true
    '''
}
```

Removes unused Docker images after pipeline execution to free disk space on the Jenkins server.

---

# Complete Deployment Flow

1. Checkout latest source code from GitHub.
2. Verify Docker, AWS CLI, and kubectl availability.
3. Authenticate Docker with Amazon Public ECR.
4. Build Quoteservice Docker image.
5. Create version-specific and latest image tags.
6. Push images to Public ECR.
7. Update Kubernetes deployment manifest.
8. Apply Deployment and Service resources.
9. Verify rollout completion.
10. Display deployment status.
11. Clean up Docker artifacts.

