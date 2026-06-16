# Checkout Service CI/CD Pipeline Documentation

## Overview

This document describes the CI/CD pipeline used to build, containerize, publish, and deploy the OpenTelemetry Demo Checkout Service to a Kubernetes cluster using Jenkins.

### Technology Stack

* Jenkins
* Docker
* AWS Public ECR
* Kubernetes
* GitHub
* OpenTelemetry Demo Application

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
Checkout Service Pod

---

# Repository Information

Repository:

https://github.com/Astronomy-NextGenIT/Open-telemetry-Mono-repo

Branch:

main

Service Path:

src/checkout

Dockerfile:

src/checkout/Dockerfile

Deployment Manifest:

src/checkout/kubernetes/checkout/deploy.yaml

Service Manifest:

src/checkout/kubernetes/checkout/svc.yaml

---

# Pipeline Configuration

## Environment Variables

| Variable       | Description                |
| -------------- | -------------------------- |
| AWS_REGION     | AWS Public ECR Region      |
| PUBLIC_ECR_URI | Public ECR Repository URI  |
| IMAGE_NAME     | Docker image name          |
| IMAGE_TAG      | Unique build tag           |
| K8S_DEPLOYMENT | Kubernetes deployment name |
| NAMESPACE      | Kubernetes namespace       |

Configured Values:

```groovy
AWS_REGION = "us-east-1"

PUBLIC_ECR_URI = "public.ecr.aws/q2a1e2p7/open-telemetry-demo"

IMAGE_NAME = "checkoutservice"

IMAGE_TAG = "${IMAGE_NAME}-${BUILD_NUMBER}"

K8S_DEPLOYMENT = "opentelemetry-demo-checkoutservice"

NAMESPACE = "default"
```

---

# Pipeline Stages

## Stage 1: Checkout Source Code

Purpose:

Pull latest application code from GitHub repository.

Pipeline Code:

```groovy
stage('Checkout') {
    steps {
        git branch: 'main',
            url: 'https://github.com/Astronomy-NextGenIT/Open-telemetry-Mono-repo.git'
    }
}
```

Outcome:

* Latest source code downloaded.
* Jenkins workspace updated.

---

## Stage 2: Verify Required Tools

Purpose:

Ensure required tools are available on Jenkins server.

Pipeline Code:

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

Verifies:

* AWS CLI
* Kubectl
* Docker

---

## Stage 3: Login to AWS Public ECR

Purpose:

Authenticate Docker client with AWS Public ECR.

Pipeline Code:

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

Outcome:

Docker can push images to Public ECR.

---

## Stage 4: Build Docker Image

Purpose:

Create Docker image for Checkout Service.

Pipeline Code:

```groovy
stage('Build Docker Image') {
    steps {
        sh '''
        docker build \
        -t ${IMAGE_NAME}:${IMAGE_TAG} \
        -f src/checkout/Dockerfile .
        '''
    }
}
```

Example:

```bash
docker build \
-t checkoutservice:checkoutservice-7 \
-f src/checkout/Dockerfile .
```

Outcome:

Docker image successfully built.

---

## Stage 5: Tag Docker Image

Purpose:

Tag image for AWS Public ECR repository.

Pipeline Code:

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

Generated Tags:

```text
public.ecr.aws/q2a1e2p7/open-telemetry-demo:checkoutservice-7

public.ecr.aws/q2a1e2p7/open-telemetry-demo:checkoutservice-latest
```

---

## Stage 6: Push Image to AWS Public ECR

Purpose:

Upload image to Public ECR.

Pipeline Code:

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

Outcome:

Image available globally through AWS Public ECR.

---

## Stage 7: Update Kubernetes Manifest

Purpose:

Replace existing image reference with newly built image.

Pipeline Code:

```groovy
stage('Update Manifest') {
    steps {
        sh '''
        sed -i "s|image:.*|image: ${PUBLIC_ECR_URI}:${IMAGE_TAG}|g" \
        src/checkout/kubernetes/checkout/deploy.yaml

        echo "Updated Image:"
        grep -n "image:" \
        src/checkout/kubernetes/checkout/deploy.yaml
        '''
    }
}
```

Example Transformation:

Before:

```yaml
image: ghcr.io/open-telemetry/demo:1.12.0-checkoutservice
```

After:

```yaml
image: public.ecr.aws/q2a1e2p7/open-telemetry-demo:checkoutservice-7
```

---

## Stage 8: Deploy to Kubernetes

Purpose:

Apply updated deployment and service manifests.

Pipeline Code:

```groovy
stage('Deploy To Kubernetes') {
    steps {
        sh '''
        kubectl apply -f src/checkout/kubernetes/checkout/deploy.yaml
        kubectl apply -f src/checkout/kubernetes/checkout/svc.yaml
        '''
    }
}
```

Outcome:

* Deployment updated
* New ReplicaSet created
* New pod scheduled

---

## Stage 9: Verify Rollout

Purpose:

Ensure deployment completes successfully.

Pipeline Code:

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

Successful Output:

```text
deployment "opentelemetry-demo-checkoutservice" successfully rolled out
```

---

# Post Actions

## Success

```groovy
success {
    echo "Checkout Service Deployment Successful"
    echo "Image Pushed: ${PUBLIC_ECR_URI}:${IMAGE_TAG}"
}
```

Displays deployment success message.

---

## Failure

```groovy
failure {
    echo "Checkout Service Pipeline Failed"
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

Removes unused Docker images and frees disk space.

---

# Complete Jenkins Pipeline

```groovy
pipeline {
    agent any

    environment {

        AWS_REGION = "us-east-1"

        PUBLIC_ECR_URI = "public.ecr.aws/q2a1e2p7/open-telemetry-demo"

        IMAGE_NAME = "checkoutservice"
        IMAGE_TAG = "${IMAGE_NAME}-${BUILD_NUMBER}"

        K8S_DEPLOYMENT = "opentelemetry-demo-checkoutservice"
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
                -f src/checkout/Dockerfile .
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
                src/checkout/kubernetes/checkout/deploy.yaml

                echo "Updated Image:"
                grep -n "image:" \
                src/checkout/kubernetes/checkout/deploy.yaml
                '''
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                sh '''
                kubectl apply -f src/checkout/kubernetes/checkout/deploy.yaml
                kubectl apply -f src/checkout/kubernetes/checkout/svc.yaml
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
            echo "Checkout Service Deployment Successful"
            echo "Image Pushed: ${PUBLIC_ECR_URI}:${IMAGE_TAG}"
        }

        failure {
            echo "Checkout Service Pipeline Failed"
        }

        always {
            sh '''
            docker image prune -af || true
            '''
        }
    }
}
```
