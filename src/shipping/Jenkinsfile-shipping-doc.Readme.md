# Shipping Service CI/CD Pipeline Documentation

## Overview

This Jenkins pipeline automates the complete CI/CD process for the Shipping Service of the OpenTelemetry Demo application. The pipeline performs source code checkout, Docker image build, image publishing to AWS Public ECR, Kubernetes manifest updates, deployment, and rollout verification.

---

# Pipeline Configuration

## Environment Variables

| Variable       | Description                                           |
| -------------- | ----------------------------------------------------- |
| AWS_REGION     | AWS region used for Public ECR authentication         |
| PUBLIC_ECR_URI | Public ECR repository URI                             |
| IMAGE_NAME     | Docker image name (shippingservice)                   |
| IMAGE_TAG      | Unique image tag generated using Jenkins build number |
| K8S_DEPLOYMENT | Kubernetes deployment name                            |
| NAMESPACE      | Kubernetes namespace                                  |

```groovy
environment {
    AWS_REGION = "us-east-1"
    PUBLIC_ECR_URI = "public.ecr.aws/q2a1e2p7/open-telemetry-demo"

    IMAGE_NAME = "shippingservice"
    IMAGE_TAG = "${IMAGE_NAME}-${BUILD_NUMBER}"

    K8S_DEPLOYMENT = "opentelemetry-demo-shippingservice"
    NAMESPACE = "default"
}
```

---

# Pipeline Stages

## Stage 1: Checkout

Pulls the latest source code from GitHub.

### Repository

```text
https://github.com/Astronomy-NextGenIT/Open-telemetry-Mono-repo.git
```

### Command

```groovy
git branch: 'main',
    url: 'https://github.com/Astronomy-NextGenIT/Open-telemetry-Mono-repo.git'
```

---

## Stage 2: Verify Tools

Verifies that required tools are installed on the Jenkins agent.

### Tools Checked

* AWS CLI
* kubectl
* Docker

### Commands

```bash
aws --version
kubectl version --client
docker --version
```

---

## Stage 3: Login to AWS Public ECR

Authenticates Docker with AWS Public Elastic Container Registry.

### Command

```bash
aws ecr-public get-login-password \
--region us-east-1 | \
docker login \
--username AWS \
--password-stdin public.ecr.aws
```

---

## Stage 4: Build Docker Image

Builds the Shipping Service Docker image.

### Dockerfile Location

```text
src/shipping/Dockerfile
```

### Command

```bash
docker build \
-t shippingservice:${BUILD_NUMBER} \
-f src/shipping/Dockerfile .
```

---

## Stage 5: Tag Docker Image

Creates versioned and latest image tags.

### Tags Created

```text
shippingservice-${BUILD_NUMBER}
shippingservice-latest
```

### Commands

```bash
docker tag shippingservice:${BUILD_NUMBER} \
public.ecr.aws/q2a1e2p7/open-telemetry-demo:shippingservice-${BUILD_NUMBER}

docker tag shippingservice:${BUILD_NUMBER} \
public.ecr.aws/q2a1e2p7/open-telemetry-demo:shippingservice-latest
```

---

## Stage 6: Push Image to Public ECR

Pushes Docker images to AWS Public ECR.

### Commands

```bash
docker push public.ecr.aws/q2a1e2p7/open-telemetry-demo:shippingservice-${BUILD_NUMBER}

docker push public.ecr.aws/q2a1e2p7/open-telemetry-demo:shippingservice-latest
```

---

## Stage 7: Update Kubernetes Manifest

Updates the deployment YAML with the newly created image tag.

### Deployment Manifest

```text
src/shipping/kubernetes/shipping/deploy.yaml
```

### Command

```bash
sed -i "s|image:.*|image: ${PUBLIC_ECR_URI}:${IMAGE_TAG}|g" \
src/shipping/kubernetes/shipping/deploy.yaml
```

### Verification

```bash
grep -n "image:" \
src/shipping/kubernetes/shipping/deploy.yaml
```

---

## Stage 8: Deploy to Kubernetes

Applies Kubernetes Deployment and Service manifests.

### Deployment

```text
src/shipping/kubernetes/shipping/deploy.yaml
```

### Service

```text
src/shipping/kubernetes/shipping/svc.yaml
```

### Commands

```bash
kubectl apply -f src/shipping/kubernetes/shipping/deploy.yaml

kubectl apply -f src/shipping/kubernetes/shipping/svc.yaml
```

---

## Stage 9: Rollout Verification

Waits until the Kubernetes deployment becomes healthy.

### Command

```bash
kubectl rollout status \
deployment/opentelemetry-demo-shippingservice \
-n default \
--timeout=300s
```

---

# Post Build Actions

## Success

Displays deployment success information.

```groovy
echo "Shipping Service Deployment Successful"
echo "Image Pushed: ${PUBLIC_ECR_URI}:${IMAGE_TAG}"
```

---

## Failure

Displays failure message.

```groovy
echo "Shipping Service Pipeline Failed"
```

---

## Cleanup

Removes unused Docker images to free disk space on Jenkins agents.

### Command

```bash
docker image prune -af || true
```

---

# Complete Jenkins Pipeline

```groovy
pipeline {
    agent any

    environment {

        AWS_REGION = "us-east-1"

        PUBLIC_ECR_URI = "public.ecr.aws/q2a1e2p7/open-telemetry-demo"

        IMAGE_NAME = "shippingservice"
        IMAGE_TAG = "${IMAGE_NAME}-${BUILD_NUMBER}"

        K8S_DEPLOYMENT = "opentelemetry-demo-shippingservice"
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
                -f src/shipping/Dockerfile .
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
                src/shipping/kubernetes/shipping/deploy.yaml

                echo "Updated Image:"
                grep -n "image:" \
                src/shipping/kubernetes/shipping/deploy.yaml
                '''
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                sh '''
                kubectl apply -f src/shipping/kubernetes/shipping/deploy.yaml
                kubectl apply -f src/shipping/kubernetes/shipping/svc.yaml
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
            echo "Shipping Service Deployment Successful"
            echo "Image Pushed: ${PUBLIC_ECR_URI}:${IMAGE_TAG}"
        }

        failure {
            echo "Shipping Service Pipeline Failed"
        }

        always {
            sh '''
            docker image prune -af || true
            '''
        }
    }
}
```
