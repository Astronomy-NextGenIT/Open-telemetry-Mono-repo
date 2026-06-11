# Currency Service CI/CD Pipeline Documentation

## Overview

This Jenkins pipeline automates the complete CI/CD workflow for the Currency Service in the OpenTelemetry Demo application. The pipeline performs source code checkout, Docker image creation, Amazon ECR image management, Kubernetes deployment, and deployment verification.

### Objectives

* Fetch latest source code from GitHub.
* Build Currency Service Docker image.
* Push image to Amazon ECR.
* Update Kubernetes deployment manifest with the latest image.
* Deploy the Currency Service to Amazon EKS.
* Verify successful rollout of the deployment.
* Clean up unused Docker images after execution.

---

# Pipeline Architecture

```text
GitHub Repository
        │
        ▼
   Jenkins Pipeline
        │
        ▼
 Build Docker Image
        │
        ▼
 Push Image to ECR
        │
        ▼
 Update Kubernetes Manifest
        │
        ▼
 Deploy to EKS
        │
        ▼
 Verify Rollout
```

---

# Environment Variables

The pipeline defines several environment variables to avoid hardcoding values and improve maintainability.

| Variable       | Description                             |
| -------------- | --------------------------------------- |
| AWS_REGION     | AWS Region where ECR and EKS are hosted |
| AWS_ACCOUNT_ID | AWS Account Number                      |
| ECR_REPO       | ECR Repository Name                     |
| IMAGE_NAME     | Docker image name for Currency Service  |
| IMAGE_TAG      | Build number used as image tag          |
| ECR_URI        | Complete ECR repository URI             |
| K8S_DEPLOYMENT | Kubernetes deployment name              |
| NAMESPACE      | Kubernetes namespace                    |

### Configuration

```groovy
environment {
    AWS_REGION = "ap-south-1"
    AWS_ACCOUNT_ID = "004058506543"

    ECR_REPO = "open-telemetry-registry"

    IMAGE_NAME = "currency"
    IMAGE_TAG = "${BUILD_NUMBER}"

    ECR_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}"

    K8S_DEPLOYMENT = "opentelemetry-demo-currencyservice"
    NAMESPACE = "default"
}
```

---
# Complete Pipeline
```groovy
pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        AWS_ACCOUNT_ID = "004058506543"

        ECR_REPO = "open-telemetry-registry"

        IMAGE_NAME = "currency"
        IMAGE_TAG = "${BUILD_NUMBER}"

        ECR_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}"

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

        stage('ECR Login') {
            steps {
                sh '''
                aws ecr get-login-password \
                --region ${AWS_REGION} | \
                docker login \
                --username AWS \
                --password-stdin \
                ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
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
                ${ECR_URI}:${IMAGE_TAG}
                '''
            }
        }

        stage('Push Image To ECR') {
            steps {
                sh '''
                docker push ${ECR_URI}:${IMAGE_TAG}
                '''
            }
        }

        stage('Update Manifest') {
            steps {
                sh '''
                sed -i "s|ghcr.io/open-telemetry/demo:1.12.0-currencyservice|${ECR_URI}:${IMAGE_TAG}|g" \
                src/currency/kubernetes/currency/deploy.yaml

                echo "Updated Image:"
                grep -n "image:" src/currency/kubernetes/currency/deploy.yaml
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
                kubectl rollout status deployment/${K8S_DEPLOYMENT} -n ${NAMESPACE}
                '''
            }
        }
    }

    post {

        success {
            echo 'Currency Service Deployment Successful'
        }

        failure {
            echo 'Pipeline Failed'
        }

        always {
            sh '''
            docker image prune -f || true
            '''
        }
    }
}

```
# Stage 1: Checkout Source Code

## Purpose

Downloads the latest source code from the GitHub repository.

### Jenkins Stage

```groovy
stage('Checkout') {
    steps {
        git branch: 'main',
        url: 'https://github.com/Astronomy-NextGenIT/Open-telemetry-Mono-repo.git'
    }
}
```

### What Happens

1. Jenkins connects to GitHub.
2. Pulls the latest code from the `main` branch.
3. Stores the repository contents inside Jenkins workspace.

### Benefits

* Ensures latest application code is used.
* Provides source files for Docker build and deployment.

---

# Stage 2: Verify Required Tools

## Purpose

Verifies that all required tools are available on the Jenkins server before proceeding.

### Jenkins Stage

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

### Tools Verified

| Tool    | Purpose                           |
| ------- | --------------------------------- |
| AWS CLI | Access AWS services               |
| kubectl | Deploy resources to Kubernetes    |
| Docker  | Build and manage container images |

### Expected Output

```bash
aws-cli/2.x.x
Client Version: v1.xx.x
Docker version xx.x.x
```

### Benefits

* Prevents failures later in the pipeline.
* Validates Jenkins agent readiness.

---

# Stage 3: Authenticate with Amazon ECR

## Purpose

Logs Jenkins into Amazon Elastic Container Registry (ECR).

### Jenkins Stage

```groovy
stage('ECR Login') {
    steps {
        sh '''
        aws ecr get-login-password \
        --region ${AWS_REGION} | \
        docker login \
        --username AWS \
        --password-stdin \
        ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
        '''
    }
}
```

### What Happens

1. AWS CLI generates a temporary authentication token.
2. Token is passed securely to Docker.
3. Docker authenticates with Amazon ECR.

### Benefits

* Allows image push operations.
* Avoids storing credentials in plaintext.

---

# Stage 4: Build Docker Image

## Purpose

Builds a Docker image for the Currency Service.

### Jenkins Stage

```groovy
stage('Build Docker Image') {
    steps {
        sh '''
        docker build \
        -t ${IMAGE_NAME}:${IMAGE_TAG} \
        -f src/currency/Dockerfile .
        '''
    }
}
```

### Docker Build Command

```bash
docker build \
-t currency:<BUILD_NUMBER> \
-f src/currency/Dockerfile .
```

### Example

If Build Number = 25

```bash
currency:25
```

### What Happens

1. Docker reads the Currency Service Dockerfile.
2. Application dependencies are installed.
3. Application is packaged into a container image.

### Benefits

* Creates a portable application package.
* Ensures consistency across environments.

---

# Stage 5: Tag Docker Image

## Purpose

Creates an ECR-compatible tag for the newly built image.

### Jenkins Stage

```groovy
stage('Tag Image') {
    steps {
        sh '''
        docker tag \
        ${IMAGE_NAME}:${IMAGE_TAG} \
        ${ECR_URI}:${IMAGE_TAG}
        '''
    }
}
```

### Example

Source Image:

```bash
currency:25
```

Target Image:

```bash
004058506543.dkr.ecr.ap-south-1.amazonaws.com/open-telemetry-registry:25
```

### Benefits

* Prepares image for ECR upload.
* Maintains version tracking using build numbers.

---

# Stage 6: Push Docker Image to Amazon ECR

## Purpose

Uploads the image to Amazon Elastic Container Registry.

### Jenkins Stage

```groovy
stage('Push Image To ECR') {
    steps {
        sh '''
        docker push ${ECR_URI}:${IMAGE_TAG}
        '''
    }
}
```

### Example

```bash
docker push 004058506543.dkr.ecr.ap-south-1.amazonaws.com/open-telemetry-registry:25
```

### What Happens

1. Docker layers are uploaded to ECR.
2. Image becomes available for Kubernetes deployment.

### Benefits

* Centralized image repository.
* Secure storage of application images.

---

# Stage 7: Update Kubernetes Deployment Manifest

## Purpose

Updates the Kubernetes deployment file with the latest Docker image.

### Jenkins Stage

```groovy
stage('Update Manifest') {
    steps {
        sh '''
        sed -i "s|ghcr.io/open-telemetry/demo:1.12.0-currencyservice|${ECR_URI}:${IMAGE_TAG}|g" \
        src/currency/kubernetes/currency/deploy.yaml

        echo "Updated Image:"
        grep -n "image:" src/currency/kubernetes/currency/deploy.yaml
        '''
    }
}
```

### Before Update

```yaml
image: ghcr.io/open-telemetry/demo:1.12.0-currencyservice
```

### After Update

```yaml
image: 004058506543.dkr.ecr.ap-south-1.amazonaws.com/open-telemetry-registry:25
```

### What Happens

* `sed` replaces the default image reference.
* Deployment manifest now points to the newly built image.
* `grep` verifies the update.

### Benefits

* Automatically deploys latest version.
* Eliminates manual manifest edits.

---

# Stage 8: Deploy Currency Service to Kubernetes

## Purpose

Applies Kubernetes resources to EKS cluster.

### Jenkins Stage

```groovy
stage('Deploy To Kubernetes') {
    steps {
        sh '''
        kubectl apply -f src/currency/kubernetes/currency/deploy.yaml
        kubectl apply -f src/currency/kubernetes/currency/svc.yaml
        '''
    }
}
```

### Resources Applied

#### Deployment

```yaml
deploy.yaml
```

Responsible for:

* Pod creation
* Replica management
* Rolling updates

#### Service

```yaml
svc.yaml
```

Responsible for:

* Internal networking
* Service discovery
* Pod communication

### Benefits

* Deploys latest application version.
* Maintains desired state automatically.

---

# Stage 9: Verify Deployment Rollout

## Purpose

Ensures the deployment has completed successfully.

### Jenkins Stage

```groovy
stage('Rollout Status') {
    steps {
        sh '''
        kubectl rollout status deployment/${K8S_DEPLOYMENT} -n ${NAMESPACE}
        '''
    }
}
```

### Executed Command

```bash
kubectl rollout status deployment/opentelemetry-demo-currencyservice -n default
```

### Successful Output

```bash
deployment "opentelemetry-demo-currencyservice" successfully rolled out
```

### What Happens

Kubernetes checks:

* New pods are created.
* Containers start successfully.
* Readiness probes pass.
* Old pods are terminated.

### Benefits

* Prevents unnoticed deployment failures.
* Validates application availability.

---

# Post Build Actions

## Success Block

```groovy
success {
    echo 'Currency Service Deployment Successful'
}
```

### Output

```bash
Currency Service Deployment Successful
```

---

## Failure Block

```groovy
failure {
    echo 'Pipeline Failed'
}
```

### Output

```bash
Pipeline Failed
```

---

## Cleanup Block

```groovy
always {
    sh '''
    docker image prune -f || true
    '''
}
```

### Purpose

Removes unused Docker images from Jenkins server.

### Benefits

* Prevents disk space exhaustion.
* Keeps Jenkins node clean.
* Reduces maintenance overhead.

---

# Complete Deployment Flow

```text
1. Checkout latest code from GitHub
            │
            ▼
2. Verify AWS CLI, Docker and kubectl
            │
            ▼
3. Login to Amazon ECR
            │
            ▼
4. Build Currency Service Docker Image
            │
            ▼
5. Tag Image for ECR
            │
            ▼
6. Push Image to Amazon ECR
            │
            ▼
7. Update Kubernetes Manifest
            │
            ▼
8. Deploy Deployment and Service
            │
            ▼
9. Verify Rollout Status
            │
            ▼
10. Cleanup Old Docker Images
```

# Outcome

After successful execution:

* Currency Service source code is fetched from GitHub.
* Docker image is built and versioned using Jenkins build number.
* Image is pushed to Amazon ECR.
* Kubernetes deployment manifest is automatically updated.
* Latest Currency Service version is deployed to Amazon EKS.
* Rollout status is validated.
* Jenkins workspace remains clean through automatic Docker image cleanup.

