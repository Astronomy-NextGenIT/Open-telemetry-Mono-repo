# Frontend Proxy CI/CD Pipeline Documentation

## Overview

This Jenkins pipeline automates the complete CI/CD process for the OpenTelemetry Demo Frontend Proxy service.

The pipeline performs the following tasks:

1. Clones the application source code from GitHub.
2. Verifies required tools on the Jenkins agent.
3. Authenticates with AWS Public ECR.
4. Builds the Docker image.
5. Tags the image with a unique build number and latest tag.
6. Pushes the image to AWS Public ECR.
7. Updates the Kubernetes deployment manifest.
8. Deploys the application to Kubernetes.
9. Verifies deployment rollout status.
10. Displays deployment information.
11. Cleans up local Docker images.

---

# Pipeline Configuration

## Environment Variables

| Variable       | Description                                    |
| -------------- | ---------------------------------------------- |
| AWS_REGION     | AWS region used for Public ECR authentication  |
| PUBLIC_ECR_URI | Public ECR repository URI                      |
| IMAGE_NAME     | Docker image name                              |
| IMAGE_TAG      | Image tag generated using Jenkins build number |
| K8S_DEPLOYMENT | Kubernetes deployment name                     |
| NAMESPACE      | Kubernetes namespace                           |

```groovy
environment {

    AWS_REGION = "us-east-1"

    PUBLIC_ECR_URI = "public.ecr.aws/q2a1e2p7/open-telemetry-demo"

    IMAGE_NAME = "frontendproxy"
    IMAGE_TAG = "${IMAGE_NAME}-${BUILD_NUMBER}"

    K8S_DEPLOYMENT = "opentelemetry-demo-frontendproxy"
    NAMESPACE = "default"
}
```

---

# Complete Jenkins Pipeline

```groovy
pipeline {
    agent any

    environment {

        AWS_REGION = "us-east-1"

        PUBLIC_ECR_URI = "public.ecr.aws/q2a1e2p7/open-telemetry-demo"

        IMAGE_NAME = "frontendproxy"
        IMAGE_TAG = "${IMAGE_NAME}-${BUILD_NUMBER}"

        K8S_DEPLOYMENT = "opentelemetry-demo-frontendproxy"
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
                -f src/frontend-proxy/Dockerfile .
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
                src/frontend-proxy/kubernetes/frontendproxy/deploy.yaml

                echo "Updated Image:"
                grep -n "image:" \
                src/frontend-proxy/kubernetes/frontendproxy/deploy.yaml
                '''
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                sh '''
                kubectl apply -f src/frontend-proxy/kubernetes/frontendproxy/deploy.yaml

                kubectl apply -f src/frontend-proxy/kubernetes/frontendproxy/svc.yaml

                kubectl apply -f src/frontend-proxy/kubernetes/frontendproxy/ingress.yaml
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

        stage('Verify Deployment') {
            steps {
                sh '''
                echo "=== Pods ==="
                kubectl get pods -o wide | grep frontendproxy || true

                echo "=== Service ==="
                kubectl get svc | grep frontendproxy || true

                echo "=== Ingress ==="
                kubectl get ingress || true
                '''
            }
        }
    }

    post {

        success {
            echo "Frontend Proxy Deployment Successful"
            echo "Image Pushed: ${PUBLIC_ECR_URI}:${IMAGE_TAG}"
        }

        failure {
            echo "Frontend Proxy Pipeline Failed"
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

## Stage 1: Checkout

### Purpose

Downloads the latest source code from GitHub.

### Command

```groovy
git branch: 'main',
    url: 'https://github.com/Astronomy-NextGenIT/Open-telemetry-Mono-repo.git'
```

### Output

Latest repository source code is downloaded to Jenkins workspace.

---

## Stage 2: Verify Tools

### Purpose

Ensures required tools are installed on Jenkins agent.

### Commands

```bash
aws --version
kubectl version --client
docker --version
```

### Verification

Checks:

* AWS CLI
* Kubectl
* Docker

---

## Stage 3: Login Public ECR

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

### Result

Jenkins can push images to Public ECR.

---

## Stage 4: Build Docker Image

### Purpose

Builds Frontend Proxy Docker image.

### Command

```bash
docker build \
-t frontendproxy:frontendproxy-${BUILD_NUMBER} \
-f src/frontend-proxy/Dockerfile .
```

### Example

```bash
frontendproxy:frontendproxy-15
```

---

## Stage 5: Tag Image

### Purpose

Creates ECR-compatible image tags.

### Commands

```bash
docker tag frontendproxy:frontendproxy-15 \
public.ecr.aws/q2a1e2p7/open-telemetry-demo:frontendproxy-15

docker tag frontendproxy:frontendproxy-15 \
public.ecr.aws/q2a1e2p7/open-telemetry-demo:frontendproxy-latest
```

### Generated Tags

```text
frontendproxy-15
frontendproxy-latest
```

---

## Stage 6: Push Image To Public ECR

### Purpose

Publishes images to Public ECR.

### Commands

```bash
docker push public.ecr.aws/q2a1e2p7/open-telemetry-demo:frontendproxy-15

docker push public.ecr.aws/q2a1e2p7/open-telemetry-demo:frontendproxy-latest
```

### Result

Images become available globally.

---

## Stage 7: Update Manifest

### Purpose

Updates Kubernetes Deployment manifest with newly built image.

### Command

```bash
sed -i "s|image:.*|image: public.ecr.aws/q2a1e2p7/open-telemetry-demo:frontendproxy-15|g"
```

### Verification

```bash
grep -n "image:" deploy.yaml
```

---

## Stage 8: Deploy To Kubernetes

### Purpose

Deploys application resources.

### Commands

```bash
kubectl apply -f deploy.yaml

kubectl apply -f svc.yaml

kubectl apply -f ingress.yaml
```

### Resources Created

* Deployment
* Service
* Ingress

---

## Stage 9: Rollout Status

### Purpose

Waits for deployment completion.

### Command

```bash
kubectl rollout status deployment/opentelemetry-demo-frontendproxy \
--timeout=300s
```

### Result

Confirms successful pod startup.

---

## Stage 10: Verify Deployment

### Purpose

Displays deployment resources.

### Commands

```bash
kubectl get pods -o wide | grep frontendproxy

kubectl get svc | grep frontendproxy

kubectl get ingress
```

### Output

Shows:

* Running Pods
* Service Information
* Ingress Configuration

---

# Post Build Actions

## Success

```groovy
success {
    echo "Frontend Proxy Deployment Successful"
    echo "Image Pushed: ${PUBLIC_ECR_URI}:${IMAGE_TAG}"
}
```

Displays successful deployment information.

---

## Failure

```groovy
failure {
    echo "Frontend Proxy Pipeline Failed"
}
```

Displays failure notification.

---

## Cleanup

```groovy
always {
    sh '''
    docker image prune -af || true
    '''
}
```

Removes unused Docker images from Jenkins agent.

---

# CI/CD Workflow Diagram

```text
GitHub Repository
        │
        ▼
 Jenkins Pipeline
        │
        ▼
 Verify Tools
        │
        ▼
 Login Public ECR
        │
        ▼
 Build Docker Image
        │
        ▼
 Tag Image
        │
        ▼
 Push To Public ECR
        │
        ▼
 Update Manifest
        │
        ▼
 Deploy To Kubernetes
        │
        ▼
 Rollout Verification
        │
        ▼
 Application Available
```

# Benefits

* Fully automated CI/CD workflow.
* Automatic image versioning using Jenkins build numbers.
* Continuous deployment to Kubernetes.
* Automated rollout validation.
* Public ECR image distribution.
* Automatic cleanup of Docker artifacts.
* Repeatable and reliable deployments.
