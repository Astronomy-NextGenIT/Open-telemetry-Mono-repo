# Kafka Service CI/CD Pipeline Documentation

## Overview

This Jenkins pipeline automates the complete CI/CD process for the Kafka service in the OpenTelemetry Demo application.

The pipeline performs the following actions:

1. Clones the source code from GitHub.
2. Verifies required tools on the Jenkins agent.
3. Authenticates with AWS Public ECR.
4. Builds the Kafka Docker image.
5. Tags the image with a build-specific version and latest tag.
6. Pushes the image to AWS Public ECR.
7. Updates the Kubernetes deployment manifest with the newly built image.
8. Deploys the Kafka service to Kubernetes.
9. Monitors rollout status.
10. Cleans up unused Docker images.

---

# Architecture Flow

GitHub Repository
↓
Jenkins Pipeline
↓
Docker Build
↓
AWS Public ECR
↓
Kubernetes Deployment
↓
Kafka Service Running

---

# Prerequisites

## Jenkins Requirements

The Jenkins server must have:

* Docker installed
* AWS CLI installed
* kubectl installed
* Access to Kubernetes cluster
* Permission to push images to AWS Public ECR

Verify:

```bash
aws --version
docker --version
kubectl version --client
```

---

## Kubernetes Files

The following files must exist:

```text
src/kafka/
├── Dockerfile
└── kubernetes/
    └── kafka/
        ├── deploy.yaml
        └── svc.yaml
```

---

# Environment Variables

| Variable                | Description                            |
| ----------------------- | -------------------------------------- |
| AWS_REGION              | AWS region used for ECR login          |
| PUBLIC_ECR_URI          | Public ECR repository URI              |
| IMAGE_NAME              | Docker image name                      |
| IMAGE_TAG               | Image tag generated using build number |
| OTEL_JAVA_AGENT_VERSION | OpenTelemetry Java Agent version       |
| K8S_DEPLOYMENT          | Kubernetes deployment name             |
| NAMESPACE               | Kubernetes namespace                   |

Configured values:

```groovy
AWS_REGION = "us-east-1"

PUBLIC_ECR_URI = "public.ecr.aws/q2a1e2p7/open-telemetry-demo"

IMAGE_NAME = "kafka"

IMAGE_TAG = "${IMAGE_NAME}-${BUILD_NUMBER}"

OTEL_JAVA_AGENT_VERSION = "2.25.0"

K8S_DEPLOYMENT = "opentelemetry-demo-kafka"

NAMESPACE = "default"
```

---

# Pipeline Stages

## Stage 1: Checkout

Downloads the latest code from GitHub.

```groovy
stage('Checkout')
```

Repository:

```text
https://github.com/Astronomy-NextGenIT/Open-telemetry-Mono-repo.git
```

---

## Stage 2: Verify Tools

Ensures required tools are available.

```groovy
stage('Verify Tools')
```

Checks:

* AWS CLI
* Docker
* kubectl

---

## Stage 3: Login Public ECR

Authenticates Jenkins with AWS Public ECR.

```groovy
stage('Login Public ECR')
```

Command:

```bash
aws ecr-public get-login-password \
--region us-east-1 | \
docker login \
--username AWS \
--password-stdin public.ecr.aws
```

---

## Stage 4: Build Docker Image

Builds Kafka image using the project Dockerfile.

```groovy
stage('Build Docker Image')
```

Docker build command:

```bash
docker build \
--build-arg OTEL_JAVA_AGENT_VERSION=2.25.0 \
-t kafka:<build-number> \
-f src/kafka/Dockerfile .
```

Generated image:

```text
kafka-1
kafka-2
kafka-3
...
```

---

## Stage 5: Tag Image

Creates two tags.

Build specific tag:

```text
public.ecr.aws/q2a1e2p7/open-telemetry-demo:kafka-<BUILD_NUMBER>
```

Latest tag:

```text
public.ecr.aws/q2a1e2p7/open-telemetry-demo:kafka-latest
```

---

## Stage 6: Push Image To Public ECR

Pushes both tags to AWS Public ECR.

```groovy
stage('Push Image To Public ECR')
```

Commands:

```bash
docker push public.ecr.aws/q2a1e2p7/open-telemetry-demo:kafka-<BUILD_NUMBER>

docker push public.ecr.aws/q2a1e2p7/open-telemetry-demo:kafka-latest
```

---

## Stage 7: Update Manifest

Updates Kubernetes deployment manifest.

File:

```text
src/kafka/kubernetes/kafka/deploy.yaml
```

Updates:

```yaml
image: public.ecr.aws/q2a1e2p7/open-telemetry-demo:kafka-<BUILD_NUMBER>
```

using:

```bash
sed -i "s|image:.*|image: NEW_IMAGE|g"
```

---

## Stage 8: Deploy To Kubernetes

Deploys Kafka resources.

```groovy
stage('Deploy To Kubernetes')
```

Commands:

```bash
kubectl apply -f src/kafka/kubernetes/kafka/deploy.yaml

kubectl apply -f src/kafka/kubernetes/kafka/svc.yaml
```

Resources created:

* Deployment
* Service

---

## Stage 9: Rollout Status

Verifies deployment success.

```groovy
stage('Rollout Status')
```

Command:

```bash
kubectl rollout status deployment/opentelemetry-demo-kafka
```

Timeout:

```text
300 seconds
```

---

# Post Actions

## Success

Displays:

```text
Kafka Deployment Successful
```

and image details.

---

## Failure

Displays:

```text
Kafka Pipeline Failed
```

---

## Cleanup

Removes unused Docker images.

```bash
docker image prune -af
```

---

# Complete Jenkins Pipeline

```groovy
pipeline {
    agent any

    environment {

        AWS_REGION = "us-east-1"

        PUBLIC_ECR_URI = "public.ecr.aws/q2a1e2p7/open-telemetry-demo"

        IMAGE_NAME = "kafka"
        IMAGE_TAG = "${IMAGE_NAME}-${BUILD_NUMBER}"

        OTEL_JAVA_AGENT_VERSION = "2.25.0"

        K8S_DEPLOYMENT = "opentelemetry-demo-kafka"
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
                -f src/kafka/Dockerfile .
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
                src/kafka/kubernetes/kafka/deploy.yaml

                grep -n "image:" \
                src/kafka/kubernetes/kafka/deploy.yaml
                '''
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                sh '''
                kubectl apply -f src/kafka/kubernetes/kafka/deploy.yaml
                kubectl apply -f src/kafka/kubernetes/kafka/svc.yaml
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
            echo "Kafka Deployment Successful"
            echo "Image Pushed: ${PUBLIC_ECR_URI}:${IMAGE_TAG}"
        }

        failure {
            echo "Kafka Pipeline Failed"
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

# Troubleshooting

## Image Pull Errors

```bash
kubectl describe pod <pod-name>
```

Verify image exists in ECR.

---

## Kafka CrashLoopBackOff

Check logs:

```bash
kubectl logs <kafka-pod> --previous
```

Common cause:

```text
KAFKA_CONTROLLER_QUORUM_VOTERS missing
```

Required:

```yaml
- name: KAFKA_CONTROLLER_QUORUM_VOTERS
  value: "1@localhost:9093"
```

---

## Deployment Verification

```bash
kubectl get pods

kubectl get deployment

kubectl get svc
```

Expected deployment:

```text
opentelemetry-demo-kafka
```
