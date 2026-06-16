# Currency Service CI/CD Pipeline Documentation 

## Overview

This Jenkins pipeline automates the complete CI/CD lifecycle for the **Currency Service** of the OpenTelemetry Demo application.

The pipeline performs the following tasks:

1. Clones source code from GitHub.
2. Verifies required tools on Jenkins.
3. Authenticates with AWS Public ECR.
4. Builds the Currency Service Docker image.
5. Tags the image with build-specific and latest tags.
6. Pushes the image to AWS Public ECR.
7. Updates the Kubernetes deployment manifest.
8. Deploys the application to Kubernetes.
9. Verifies successful rollout.
10. Cleans up unused Docker images.

---

# Architecture Flow

```text
GitHub Repository
       │
       ▼
    Jenkins
       │
       ├── Clone Source Code
       │
       ├── Build Docker Image
       │
       ├── Push Image
       ▼
 AWS Public ECR
       │
       ▼
 Kubernetes Cluster
       │
       ▼
 Currency Service Deployment
```

---

# Pipeline Variables

```groovy
environment {

    AWS_REGION = "us-east-1"

    PUBLIC_ECR_URI = "public.ecr.aws/q2a1e2p7/open-telemetry-demo"

    IMAGE_NAME = "currencyservice"
    IMAGE_TAG = "${IMAGE_NAME}-${BUILD_NUMBER}"

    K8S_DEPLOYMENT = "opentelemetry-demo-currencyservice"
    NAMESPACE = "default"
}
```

| Variable       | Purpose                     |
| -------------- | --------------------------- |
| AWS_REGION     | AWS Public ECR Region       |
| PUBLIC_ECR_URI | Public ECR repository URI   |
| IMAGE_NAME     | Currency Service image name |
| IMAGE_TAG      | Unique build tag            |
| K8S_DEPLOYMENT | Kubernetes deployment name  |
| NAMESPACE      | Kubernetes namespace        |

---

# Complete Jenkins Pipeline

```groovy
pipeline {
    agent any

    environment {

        AWS_REGION = "us-east-1"

        PUBLIC_ECR_URI = "public.ecr.aws/q2a1e2p7/open-telemetry-demo"

        IMAGE_NAME = "currencyservice"
        IMAGE_TAG = "${IMAGE_NAME}-${BUILD_NUMBER}"

        K8S_DEPLOYMENT = "opentelemetry-demo-currencyservice"
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
                -f src/currency/Dockerfile .
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
                src/currency/kubernetes/currency/deploy.yaml

                echo "Updated Image:"
                grep -n "image:" \
                src/currency/kubernetes/currency/deploy.yaml
                '''
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                sh '''
                kubectl apply -f src/currency/kubernetes/currency/deploy.yaml
                kubectl apply -f src/currency/kubernetes/currency/svc.yaml
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
            echo "Currency Service Deployment Successful"
            echo "Image Pushed: ${PUBLIC_ECR_URI}:${IMAGE_TAG}"
        }

        failure {
            echo "Currency Service Pipeline Failed"
        }

        always {
            sh '''
            docker image prune -af || true
            '''
        }
    }
}
```

---

# Stage-by-Stage Explanation

## Stage 1: Checkout Source Code

```groovy
stage('Checkout')
```

Clones the latest source code from GitHub.

```bash
git clone
```

Repository:

```text
https://github.com/Astronomy-NextGenIT/Open-telemetry-Mono-repo.git
```

Branch:

```text
main
```

---

## Stage 2: Verify Tools

```groovy
stage('Verify Tools')
```

Checks availability of:

```bash
aws --version
kubectl version --client
docker --version
```

Purpose:

* Ensure Jenkins node is properly configured.
* Prevent failures later in the pipeline.

---

## Stage 3: Login to AWS Public ECR

```groovy
stage('Login Public ECR')
```

Authenticates Docker with AWS Public ECR.

```bash
aws ecr-public get-login-password
```

Docker login:

```bash
docker login public.ecr.aws
```

Purpose:

* Allow image push operations.

---

## Stage 4: Build Docker Image

```groovy
stage('Build Docker Image')
```

Builds Currency Service image.

Dockerfile:

```text
src/currency/Dockerfile
```

Build command:

```bash
docker build \
-t currencyservice:currencyservice-${BUILD_NUMBER} \
-f src/currency/Dockerfile .
```

Example:

```bash
currencyservice:currencyservice-12
```

---

## Stage 5: Tag Docker Image

```groovy
stage('Tag Image')
```

Creates two tags.

### Versioned Tag

```bash
public.ecr.aws/q2a1e2p7/open-telemetry-demo:currencyservice-12
```

### Latest Tag

```bash
public.ecr.aws/q2a1e2p7/open-telemetry-demo:currencyservice-latest
```

Benefits:

* Rollback support.
* Latest deployment support.

---

## Stage 6: Push Image to Public ECR

```groovy
stage('Push Image To Public ECR')
```

Pushes both tags.

```bash
docker push currencyservice-12
docker push currencyservice-latest
```

Result:

```text
public.ecr.aws/q2a1e2p7/open-telemetry-demo
```

contains:

```text
currencyservice-12
currencyservice-latest
```

---

## Stage 7: Update Kubernetes Manifest

```groovy
stage('Update Manifest')
```

Replaces image reference inside deployment manifest.

Before:

```yaml
image: ghcr.io/open-telemetry/demo:1.12.0-currencyservice
```

After:

```yaml
image: public.ecr.aws/q2a1e2p7/open-telemetry-demo:currencyservice-12
```

Command:

```bash
sed -i
```

Verification:

```bash
grep image:
```

---

## Stage 8: Deploy to Kubernetes

```groovy
stage('Deploy To Kubernetes')
```

Applies Kubernetes manifests.

Deployment:

```bash
kubectl apply -f deploy.yaml
```

Service:

```bash
kubectl apply -f svc.yaml
```

Files:

```text
src/currency/kubernetes/currency/deploy.yaml
src/currency/kubernetes/currency/svc.yaml
```

---

## Stage 9: Verify Rollout

```groovy
stage('Rollout Status')
```

Monitors deployment progress.

Command:

```bash
kubectl rollout status deployment/opentelemetry-demo-currencyservice
```

Timeout:

```bash
300 seconds
```

Successful output:

```text
deployment successfully rolled out
```

---

# Post Build Actions

## Success

```groovy
success {
    echo "Currency Service Deployment Successful"
}
```

Output:

```text
Currency Service Deployment Successful
Image Pushed:
public.ecr.aws/q2a1e2p7/open-telemetry-demo:currencyservice-<build-number>
```

---

## Failure

```groovy
failure {
    echo "Currency Service Pipeline Failed"
}
```

Displays failure message for troubleshooting.

---

## Cleanup

```groovy
always {
    docker image prune -af
}
```

Removes unused Docker images from Jenkins node.

Benefits:

* Saves disk space.
* Prevents Jenkins node storage exhaustion.

---

# Expected Outcome

After successful pipeline execution:

Currency Service image built successfully
Image pushed to AWS Public ECR
Kubernetes deployment updated
New Currency Service pod created
Rollout completed successfully
Application available inside the OpenTelemetry Demo environment

✅ Jenkins workspace cleaned automatically after build completion
