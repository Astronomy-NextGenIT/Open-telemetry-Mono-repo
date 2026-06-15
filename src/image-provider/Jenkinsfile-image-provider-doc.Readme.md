# ImageProvider CI/CD Pipeline Documentation

## Overview

This document describes the CI/CD pipeline for the OpenTelemetry Demo ImageProvider service.

The pipeline performs the following operations:

1. Clones the source code from GitHub.
2. Builds the Docker image for the ImageProvider service.
3. Authenticates with AWS Public ECR.
4. Pushes the Docker image to Public ECR.
5. Updates the Kubernetes deployment with the newly built image.
6. Verifies the deployment rollout.

---

# Service Information

### Service Name

ImageProvider

### Repository Path

```text
src/image-provider
```

### Kubernetes Deployment

```text
opentelemetry-demo-imageprovider
```

### Container Name

```text
imageprovider
```

### Namespace

```text
default
```

### Dockerfile Location

```text
src/image-provider/Dockerfile
```

---

# Architecture Flow

```text
GitHub Repository
        │
        ▼
Jenkins Pipeline
        │
        ▼
Docker Build
        │
        ▼
AWS Public ECR
        │
        ▼
Kubernetes Deployment Update
        │
        ▼
New Pod Rollout
```

---

# Jenkins Pipeline

```groovy
pipeline {
    agent any

    environment {

        AWS_REGION = "us-east-1"

        PUBLIC_ECR_URI = "public.ecr.aws/q2a1e2p7/open-telemetry-demo"

        IMAGE_NAME = "imageprovider"
        IMAGE_TAG = "${IMAGE_NAME}-${BUILD_NUMBER}"

        K8S_DEPLOYMENT = "opentelemetry-demo-imageprovider"
        NAMESPACE = "default"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Astronomy-NextGenIT/Open-telemetry-Mono-repo.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build \
                  -t ${PUBLIC_ECR_URI}:${IMAGE_TAG} \
                  -f src/image-provider/Dockerfile \
                  .
                """
            }
        }

        stage('Login to Public ECR') {
            steps {
                sh '''
                aws ecr-public get-login-password \
                --region us-east-1 | \
                docker login \
                --username AWS \
                --password-stdin public.ecr.aws
                '''
            }
        }

        stage('Push Image') {
            steps {
                sh """
                docker push ${PUBLIC_ECR_URI}:${IMAGE_TAG}
                """
            }
        }

        stage('Update Deployment') {
            steps {
                sh """
                kubectl set image deployment/${K8S_DEPLOYMENT} \
                imageprovider=${PUBLIC_ECR_URI}:${IMAGE_TAG} \
                -n ${NAMESPACE}

                kubectl rollout status deployment/${K8S_DEPLOYMENT} \
                -n ${NAMESPACE}
                """
            }
        }
    }

    post {
        success {
            echo "ImageProvider deployment updated successfully"
        }

        failure {
            echo "ImageProvider deployment failed"
        }
    }
}
```

---

# Stage-by-Stage Explanation

## Stage 1: Checkout

Purpose:

Clone the latest source code from GitHub.

Command:

```groovy
git branch: 'main',
url: 'https://github.com/Astronomy-NextGenIT/Open-telemetry-Mono-repo.git'
```

Output:

```text
Workspace populated with latest code
```

---

## Stage 2: Build Docker Image

Purpose:

Build the ImageProvider Docker image.

Command:

```bash
docker build \
-t public.ecr.aws/q2a1e2p7/open-telemetry-demo:imageprovider-${BUILD_NUMBER} \
-f src/image-provider/Dockerfile .
```

Generated Image Example:

```text
public.ecr.aws/q2a1e2p7/open-telemetry-demo:imageprovider-12
```

---

## Stage 3: Login to Public ECR

Purpose:

Authenticate Jenkins to AWS Public ECR.

Command:

```bash
aws ecr-public get-login-password \
--region us-east-1 | \
docker login \
--username AWS \
--password-stdin public.ecr.aws
```

Expected Result:

```text
Login Succeeded
```

---

## Stage 4: Push Image

Purpose:

Push the newly built image to Public ECR.

Command:

```bash
docker push public.ecr.aws/q2a1e2p7/open-telemetry-demo:imageprovider-${BUILD_NUMBER}
```

Expected Result:

```text
Pushed image successfully
```

---

## Stage 5: Update Kubernetes Deployment

Purpose:

Update the running deployment with the new image.

Command:

```bash
kubectl set image deployment/opentelemetry-demo-imageprovider \
imageprovider=public.ecr.aws/q2a1e2p7/open-telemetry-demo:imageprovider-${BUILD_NUMBER}
```

This updates:

```text
Container Name: imageprovider
Deployment Name: opentelemetry-demo-imageprovider
```

---

## Rollout Verification

Command:

```bash
kubectl rollout status deployment/opentelemetry-demo-imageprovider
```

Expected Output:

```text
deployment "opentelemetry-demo-imageprovider" successfully rolled out
```

---

# Validation Commands

## Verify Deployment Image

```bash
kubectl describe deployment opentelemetry-demo-imageprovider | grep Image
```

Expected:

```text
Image: public.ecr.aws/q2a1e2p7/open-telemetry-demo:imageprovider-<build-number>
```

---

## Verify Pod Status

```bash
kubectl get pods | grep imageprovider
```

Expected:

```text
1/1 Running
```

---

## Verify Rollout History

```bash
kubectl rollout history deployment/opentelemetry-demo-imageprovider
```

---

# Troubleshooting

## Docker Build Failure

Check:

```bash
src/image-provider/Dockerfile
```

Verify path exists:

```bash
ls src/image-provider
```

---

## ECR Login Failure

Verify AWS credentials:

```bash
aws sts get-caller-identity
```

---

## Push Failure

Verify repository URI:

```text
public.ecr.aws/q2a1e2p7/open-telemetry-demo
```

---

## Kubernetes Update Failure

Verify deployment exists:

```bash
kubectl get deployment opentelemetry-demo-imageprovider
```

Verify container name:

```bash
kubectl describe deployment opentelemetry-demo-imageprovider
```

Expected:

```text
Containers:
  imageprovider:
```

---

# Success Criteria

The pipeline is considered successful when:

1. Docker image builds successfully.
2. Image is pushed to Public ECR.
3. Kubernetes deployment image is updated.
4. Rollout completes successfully.
5. Pod reaches Running state.
6. Deployment uses the newly generated image tag.

