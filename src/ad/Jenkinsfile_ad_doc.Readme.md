# Ad Service CI/CD Pipeline Documentation

## Overview

This Jenkins Pipeline automates the complete CI/CD workflow for the OpenTelemetry Demo Ad Service. The pipeline performs source code checkout, Docker image build, image push to AWS Public ECR, Kubernetes deployment update, and rollout verification.

The pipeline ensures that every build produces a uniquely tagged Docker image and deploys the latest version to the Kubernetes cluster.

---

# Pipeline Architecture

GitHub Repository
↓
Jenkins Pipeline
↓
Docker Build
↓
AWS Public ECR
↓
Kubernetes Deployment Update
↓
Kubernetes Cluster
↓
Rollout Verification

---

# Complete Jenkins Pipeline

```groovy
pipeline {
    agent any

    environment {

        AWS_REGION = "us-east-1"

        PUBLIC_ECR_URI = "public.ecr.aws/q2a1e2p7/open-telemetry-demo"

        IMAGE_NAME = "ad"
        IMAGE_TAG = "${IMAGE_NAME}-${BUILD_NUMBER}"

        K8S_DEPLOYMENT = "opentelemetry-demo-adservice"
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
                --build-arg OTEL_JAVA_AGENT_VERSION=2.25.0 \
                -t ${IMAGE_NAME}:${IMAGE_TAG} \
                -f src/ad/Dockerfile .
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
                src/ad/kubernetes/ad/deploy.yaml

                echo "Updated Image:"
                grep -n "image:" \
                src/ad/kubernetes/ad/deploy.yaml
                '''
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                sh '''
                kubectl apply -f src/ad/kubernetes/ad/deploy.yaml
                kubectl apply -f src/ad/kubernetes/ad/svc.yaml
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
            echo "Ad Service Deployment Successful"
            echo "Image Pushed: ${PUBLIC_ECR_URI}:${IMAGE_TAG}"
        }

        failure {
            echo "Ad Service Pipeline Failed"
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

# Environment Variables

| Variable       | Description                                            |
| -------------- | ------------------------------------------------------ |
| AWS_REGION     | AWS region used for Public ECR authentication          |
| PUBLIC_ECR_URI | AWS Public ECR repository URI                          |
| IMAGE_NAME     | Docker image name for Ad Service                       |
| IMAGE_TAG      | Unique image tag generated using Jenkins build number  |
| K8S_DEPLOYMENT | Kubernetes deployment name                             |
| NAMESPACE      | Kubernetes namespace where the application is deployed |

---

# Stage-by-Stage Explanation

## 1. Checkout

### Purpose

Downloads the latest source code from the GitHub repository.

### Command

```bash
git clone repository
```

### Outcome

Latest application code is available inside the Jenkins workspace.

---

## 2. Verify Tools

### Purpose

Verifies required tools are installed on the Jenkins agent.

### Tools Verified

* AWS CLI
* kubectl
* Docker

### Commands

```bash
aws --version
kubectl version --client
docker --version
```

### Outcome

Ensures build agent is properly configured before proceeding.

---

## 3. Login Public ECR

### Purpose

Authenticates Docker with AWS Public Elastic Container Registry.

### Command

```bash
aws ecr-public get-login-password \
--region us-east-1 | \
docker login \
--username AWS \
--password-stdin public.ecr.aws
```

### Outcome

Docker can push images to AWS Public ECR.

---

## 4. Build Docker Image

### Purpose

Builds the Ad Service Docker image.

### Command

```bash
docker build \
--build-arg OTEL_JAVA_AGENT_VERSION=2.25.0 \
-t ad:ad-${BUILD_NUMBER} \
-f src/ad/Dockerfile .
```

### Important Note

The Dockerfile contains:

```dockerfile
ARG OTEL_JAVA_AGENT_VERSION
```

The OpenTelemetry Java Agent is downloaded during image build. Therefore the build argument must be supplied.

### Outcome

Docker image is successfully built.

---

## 5. Tag Image

### Purpose

Creates tags for versioned and latest images.

### Tags Created

```text
public.ecr.aws/q2a1e2p7/open-telemetry-demo:ad-<BUILD_NUMBER>

public.ecr.aws/q2a1e2p7/open-telemetry-demo:ad-latest
```

### Outcome

Images are ready for upload.

---

## 6. Push Image To Public ECR

### Purpose

Uploads Docker images to AWS Public ECR.

### Commands

```bash
docker push image-tag
docker push image-latest
```

### Outcome

Images become publicly accessible from AWS Public ECR.

---

## 7. Update Manifest

### Purpose

Updates Kubernetes deployment manifest with the newly built image.

### Command

```bash
sed -i "s|image:.*|image: NEW_IMAGE|g"
```

### Example

Before:

```yaml
image: public.ecr.aws/q2a1e2p7/open-telemetry-demo:ad-latest
```

After:

```yaml
image: public.ecr.aws/q2a1e2p7/open-telemetry-demo:ad-15
```

### Outcome

Deployment manifest points to the latest build image.

---

## 8. Deploy To Kubernetes

### Purpose

Applies updated Kubernetes resources.

### Commands

```bash
kubectl apply -f deploy.yaml
kubectl apply -f svc.yaml
```

### Resources Updated

* Deployment
* Service

### Outcome

Kubernetes receives updated configuration.

---

## 9. Rollout Status

### Purpose

Waits until Kubernetes completes deployment rollout.

### Command

```bash
kubectl rollout status deployment/opentelemetry-demo-adservice
```

### Outcome

Confirms successful deployment and pod readiness.

---

# Post Actions

## Success

Displays deployment success message.

```text
Ad Service Deployment Successful
```

---

## Failure

Displays pipeline failure message.

```text
Ad Service Pipeline Failed
```

---

## Always

Removes unused Docker images.

```bash
docker image prune -af
```

This helps prevent Jenkins server disk exhaustion.

---

# Expected Workflow

1. Developer pushes code to GitHub.
2. Jenkins pipeline starts.
3. Source code is downloaded.
4. Docker image is built.
5. Image is tagged.
6. Image is pushed to AWS Public ECR.
7. Kubernetes deployment manifest is updated.
8. Application is deployed to Kubernetes.
9. Rollout verification is performed.
10. Success or failure notification is displayed.

---



```
```
