# Fraud Detection Service CI/CD Pipeline Documentation

## Overview

This Jenkins pipeline automates the complete CI/CD process for the Fraud Detection Service in the OpenTelemetry Demo application. The pipeline performs source code checkout, Docker image build, image push to Amazon Public ECR, and deployment to Kubernetes.

Unlike the Email Service pipeline, this implementation uses `kubectl set image` to update the deployment image directly instead of modifying Kubernetes manifest files. This approach prevents accidental changes to other containers such as init containers and provides a safer deployment strategy.

---

# Pipeline Workflow

The pipeline executes the following stages:

1. Checkout Source Code
2. Verify Required Tools
3. Login to Amazon Public ECR
4. Build Docker Image
5. Tag Docker Image
6. Push Docker Image to Public ECR
7. Update Kubernetes Deployment Image
8. Verify Rollout Status
9. Cleanup Docker Images

---

# Environment Variables

The pipeline defines common variables in the environment block.

```groovy
AWS_REGION = "us-east-1"
PUBLIC_ECR_URI = "public.ecr.aws/q2a1e2p7/open-telemetry-demo"

IMAGE_NAME = "frauddetectionservice"
IMAGE_TAG = "${IMAGE_NAME}-${BUILD_NUMBER}"

OTEL_JAVA_AGENT_VERSION = "2.25.0"

K8S_DEPLOYMENT = "opentelemetry-demo-frauddetectionservice"
NAMESPACE = "default"
```

## Variable Description

### AWS_REGION

Specifies the AWS region used for Public ECR authentication.

Example:

```text
us-east-1
```

---

### PUBLIC_ECR_URI

Amazon Public ECR repository where Docker images are stored.

Example:

```text
public.ecr.aws/q2a1e2p7/open-telemetry-demo
```

---

### IMAGE_NAME

Defines the Docker image name.

Example:

```text
frauddetectionservice
```

---

### IMAGE_TAG

Creates a unique image tag using the Jenkins build number.

Example:

```text
frauddetectionservice-1
frauddetectionservice-2
frauddetectionservice-3
```

This ensures every build creates a unique image version.

---

### OTEL_JAVA_AGENT_VERSION

Defines the OpenTelemetry Java Agent version used during Docker image build.

Example:

```text
2.25.0
```

This variable is passed as a Docker build argument.

---

### K8S_DEPLOYMENT

Target Kubernetes deployment name.

Example:

```text
opentelemetry-demo-frauddetectionservice
```

---

### NAMESPACE

Kubernetes namespace where the deployment exists.

Example:

```text
default
```

---

# Complete pipeline 

```groovy
pipeline {
    agent any

    environment {

        AWS_REGION = "us-east-1"

        PUBLIC_ECR_URI = "public.ecr.aws/q2a1e2p7/open-telemetry-demo"

        IMAGE_NAME = "frauddetectionservice"
        IMAGE_TAG = "${IMAGE_NAME}-${BUILD_NUMBER}"

        OTEL_JAVA_AGENT_VERSION = "2.25.0"

        K8S_DEPLOYMENT = "opentelemetry-demo-frauddetectionservice"
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
                --build-arg OTEL_JAVA_AGENT_VERSION=${OTEL_JAVA_AGENT_VERSION} \
                -t ${IMAGE_NAME}:${IMAGE_TAG} \
                -f src/fraud-detection/Dockerfile .
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

        stage('Update Deployment Image') {
            steps {
                sh '''
                kubectl set image \
                deployment/${K8S_DEPLOYMENT} \
                frauddetectionservice=${PUBLIC_ECR_URI}:${IMAGE_TAG}

                echo "Deployment updated to:"
                kubectl get deployment ${K8S_DEPLOYMENT} \
                -o=jsonpath='{.spec.template.spec.containers[0].image}'

                echo
                '''
            }
        }

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
    }

    post {

        success {
            echo "Fraud Detection Deployment Successful"
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
# Stage 1: Checkout Source Code

```groovy
stage('Checkout')
```

Purpose:

Pulls the latest application source code from GitHub.

Command:

```groovy
git branch: 'main',
url: 'https://github.com/Astronomy-NextGenIT/Open-telemetry-Mono-repo.git'
```

Outcome:

Jenkins workspace contains the latest Fraud Detection Service code.

---

# Stage 2: Verify Tools

```groovy
stage('Verify Tools')
```

Purpose:

Verifies required tools are installed on the Jenkins server.

Commands:

```bash
aws --version
kubectl version --client
docker --version
```

Outcome:

Confirms AWS CLI, kubectl, and Docker are available before proceeding.

---

# Stage 3: Login to Public ECR

```groovy
stage('Login Public ECR')
```

Purpose:

Authenticates Docker with Amazon Public ECR.

Command:

```bash
aws ecr-public get-login-password \
--region us-east-1 | \
docker login \
--username AWS \
--password-stdin public.ecr.aws
```

Outcome:

Docker gains permission to push images into Public ECR.

---

# Stage 4: Build Docker Image

```groovy
stage('Build Docker Image')
```

Purpose:

Builds the Fraud Detection Service Docker image.

Command:

```bash
docker build \
--build-arg OTEL_JAVA_AGENT_VERSION=2.25.0 \
-t frauddetectionservice:<build-number> \
-f src/fraud-detection/Dockerfile .
```

Special Note:

The Fraud Detection Dockerfile downloads the OpenTelemetry Java Agent using:

```dockerfile
ARG OTEL_JAVA_AGENT_VERSION
```

Without this build argument the build fails with:

```text
404 Invalid Response Status
```

Outcome:

A local Docker image is created.

Example:

```text
frauddetectionservice:4
```

---

# Stage 5: Tag Docker Image

```groovy
stage('Tag Image')
```

Purpose:

Creates tags for pushing images to Public ECR.

Commands:

```bash
docker tag frauddetectionservice:4 \
public.ecr.aws/q2a1e2p7/open-telemetry-demo:frauddetectionservice-4

docker tag frauddetectionservice:4 \
public.ecr.aws/q2a1e2p7/open-telemetry-demo:frauddetectionservice-latest
```

Outcome:

Creates both versioned and latest tags.

---

# Stage 6: Push Docker Image to Public ECR

```groovy
stage('Push Image To Public ECR')
```

Purpose:

Uploads Docker images to Amazon Public ECR.

Commands:

```bash
docker push public.ecr.aws/q2a1e2p7/open-telemetry-demo:frauddetectionservice-4

docker push public.ecr.aws/q2a1e2p7/open-telemetry-demo:frauddetectionservice-latest
```

Outcome:

Images become available for Kubernetes deployment.

---

# Stage 7: Update Kubernetes Deployment Image

```groovy
stage('Update Deployment Image')
```

Purpose:

Updates only the Fraud Detection application container image.

Command:

```bash
kubectl set image \
deployment/opentelemetry-demo-frauddetectionservice \
frauddetectionservice=public.ecr.aws/q2a1e2p7/open-telemetry-demo:frauddetectionservice-4
```

Verification:

```bash
kubectl get deployment \
opentelemetry-demo-frauddetectionservice \
-o=jsonpath='{.spec.template.spec.containers[0].image}'
```

---

## Why kubectl set image was used

Initially the pipeline used:

```bash
sed -i "s|image:.*|image:new-image|g"
```

This caused an issue because it replaced every image reference in the manifest.

Example:

```yaml
containers:
  image: frauddetectionservice

initContainers:
  image: busybox
```

Both images were replaced.

Result:

```yaml
initContainers:
  image: frauddetectionservice
```

BusyBox commands:

```bash
sh
nc
```

were unavailable, causing:

```text
Init:CrashLoopBackOff
```

Using:

```bash
kubectl set image
```

updates only the application container and leaves the BusyBox init container unchanged.

This is the recommended production approach.

---

# Stage 8: Rollout Status

```groovy
stage('Rollout Status')
```

Purpose:

Monitors deployment progress.

Command:

```bash
kubectl rollout status \
deployment/opentelemetry-demo-frauddetectionservice \
--timeout=300s
```

Outcome:

Pipeline waits until deployment becomes healthy.

Success Example:

```text
deployment "opentelemetry-demo-frauddetectionservice" successfully rolled out
```

---

# Post Actions

## Success

```groovy
success {
    echo "Fraud Detection Deployment Successful"
}
```

Displays deployment success message.

---

## Failure

```groovy
failure {
    echo "Pipeline Failed"
}
```

Displays failure message.

---

## Cleanup

```groovy
always {
    docker image prune -af
}
```

Purpose:

Removes unused Docker images from Jenkins server.

Benefits:

* Saves disk space
* Prevents image accumulation
* Keeps Jenkins node healthy

---

# Key Issues Resolved During Implementation

## Issue 1

Docker Build Failure

Error:

```text
404 Invalid Response Status
```

Cause:

Missing OTEL_JAVA_AGENT_VERSION build argument.

Solution:

```bash
--build-arg OTEL_JAVA_AGENT_VERSION=2.25.0
```

---

## Issue 2

Init Container CrashLoopBackOff

Cause:

sed command replaced BusyBox image.

Solution:

Replaced manifest editing with:

```bash
kubectl set image
```

---

## Issue 3

Fraud Detection Service Crash

Error:

```text
KAFKA_ADDR is not supplied
```

Cause:

Deployment provided:

```yaml
KAFKA_SERVICE_ADDR
```

Application expected:

```yaml
KAFKA_ADDR
```

Solution:

Update deployment manifest to use:

```yaml
KAFKA_ADDR=opentelemetry-demo-kafka:9092
```

---

# Final Outcome

The pipeline now provides a complete automated CI/CD workflow that:

* Builds the Fraud Detection Service
* Pushes images to Amazon Public ECR
* Updates Kubernetes deployments safely
* Avoids init container modification issues
* Supports versioned image deployments
* Performs automated rollout verification
* Cleans up build artifacts after execution

