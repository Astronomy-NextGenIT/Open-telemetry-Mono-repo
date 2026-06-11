# Jenkins CI/CD Pipeline Documentation for Cart Service Deployment

## 1. Purpose

This document describes the complete CI/CD pipeline implementation for the Cart Service component of the OpenTelemetry Demo application.

The pipeline automates the following activities:

1. Fetch source code from Git repository.
2. Build a Docker image for Cart Service.
3. Authenticate with Amazon Elastic Container Registry (ECR).
4. Push the Docker image to ECR.
5. Update Kubernetes deployment manifests with the new image.
6. Deploy the updated application to Kubernetes.
7. Verify successful rollout.
8. Perform post-build cleanup.

---

# 2. Architecture Overview

Developer Code Commit

↓

Git Repository

↓

Jenkins Pipeline

↓

Docker Image Build

↓

Amazon ECR

↓

Kubernetes Deployment Update

↓

Application Running on Kubernetes Cluster

---

# 3. Repository Structure

The pipeline expects the following structure inside the Git repository.

```text
src/
 └── cart/
      ├── src/
      │    └── Dockerfile
      └── kubernetes/
           └── cart/
                ├── deploy.yaml
                └── svc.yaml
```

Files:

Dockerfile

* Builds Cart Service image.

deploy.yaml

* Kubernetes Deployment definition.

svc.yaml

* Kubernetes Service definition.

---
# Complete Jenkinsfile

```groovy
pipeline {
    agent any

    environment {

        AWS_REGION = "ap-south-1"
        AWS_ACCOUNT_ID = "0040585065890"

        ECR_REPO = "open-telemetry-registry"

        IMAGE_NAME = "cart"

        IMAGE_TAG = "${BUILD_NUMBER}"

        ECR_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}"

        K8S_DEPLOYMENT = "opentelemetry-demo-cartservice"
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
                -f src/cart/src/Dockerfile .
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
                sed -i "s|ghcr.io/open-telemetry/demo:1.12.0-cartservice|${ECR_URI}:${IMAGE_TAG}|g" \
                src/cart/kubernetes/cart/deploy.yaml

                echo "Updated Images:"
                grep -n "image:" src/cart/kubernetes/cart/deploy.yaml
                '''
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                sh '''
                kubectl apply -f src/cart/kubernetes/cart/deploy.yaml
                kubectl apply -f src/cart/kubernetes/cart/svc.yaml
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
            echo 'Cart Service Deployment Successful'
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

# 4. Jenkins Pipeline Configuration

## Environment Variables

| Variable       | Value                          |
| -------------- | ------------------------------ |
| AWS_REGION     | ap-south-1                     |
| AWS_ACCOUNT_ID | 0040585065890                  |
| ECR_REPO       | open-telemetry-registry        |
| IMAGE_NAME     | cart                           |
| IMAGE_TAG      | BUILD_NUMBER                   |
| K8S_DEPLOYMENT | opentelemetry-demo-cartservice |
| NAMESPACE      | default                        |

---

# 5. Pipeline Stages

## Stage 1: Checkout Source Code

### Objective

Retrieve the latest Cart Service source code from GitHub.

### Pipeline Code

```groovy
stage('Checkout') {
    steps {
        git branch: 'main',
        url: 'https://github.com/Astronomy-NextGenIT/Open-telemetry-Mono-repo.git'
    }
}
```

### Execution Flow

1. Jenkins connects to GitHub.
2. Main branch is cloned.
3. Source code becomes available in Jenkins workspace.

### Expected Output

```text
Checking out Revision xxxx
Branch main
Finished cloning repository
```

---

## Stage 2: Verify Tools

### Objective

Verify that all required tools are installed.

### Tools Verified

| Tool    | Purpose               |
| ------- | --------------------- |
| AWS CLI | ECR Authentication    |
| Docker  | Image Build & Push    |
| kubectl | Kubernetes Deployment |

### Pipeline Code

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

### Expected Output

```text
aws-cli/2.x.x
Client Version: v1.30.x
Docker version 27.x.x
```

---

## Stage 3: Amazon ECR Authentication

### Objective

Authenticate Jenkins with Amazon ECR.

### Pipeline Code

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

### Execution Flow

1. AWS generates temporary login token.
2. Docker authenticates with ECR.
3. Secure image push access is established.

### Expected Output

```text
Login Succeeded
```

---

## Stage 4: Build Docker Image

### Objective

Build the Cart Service Docker image.

### Pipeline Code

```groovy
stage('Build Docker Image') {
    steps {
        sh '''
        docker build \
        -t ${IMAGE_NAME}:${IMAGE_TAG} \
        -f src/cart/src/Dockerfile .
        '''
    }
}
```

### Build Inputs

| Input       | Description                   |
| ----------- | ----------------------------- |
| Dockerfile  | Cart Service Dockerfile       |
| Source Code | Cart Service Application Code |

### Execution Flow

1. Docker reads Dockerfile.
2. Downloads required base image.
3. Copies application source.
4. Builds image layers.
5. Creates final container image.

### Example Image

```text
cart:25
```

### Expected Output

```text
Successfully built abc123
Successfully tagged cart:25
```

---

## Stage 5: Tag Docker Image

### Objective

Prepare image for Amazon ECR.

### Pipeline Code

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

Before:

```text
cart:25
```

After:

```text
004058506543.dkr.ecr.ap-south-1.amazonaws.com/open-telemetry-registry:25
```

### Importance

* Required before pushing image to ECR.
* Associates image with correct repository.

---

## Stage 6: Push Image To ECR

### Objective

Upload Cart Service image to Amazon ECR.

### Pipeline Code

```groovy
stage('Push Image To ECR') {
    steps {
        sh '''
        docker push ${ECR_URI}:${IMAGE_TAG}
        '''
    }
}
```

### Execution Flow

1. Docker connects to ECR.
2. Uploads image layers.
3. Creates image manifest.
4. Stores image in repository.

### Expected Output

```text
Layer already exists
Pushed
digest: sha256:xxxx
```

---

## Stage 7: Update Kubernetes Manifest

### Objective

Replace old Cart Service image with newly built image.

### Pipeline Code

```groovy
stage('Update Manifest') {
    steps {
        sh '''
        sed -i "s|ghcr.io/open-telemetry/demo:1.12.0-cartservice|${ECR_URI}:${IMAGE_TAG}|g" \
        src/cart/kubernetes/cart/deploy.yaml

        grep -n "image:" src/cart/kubernetes/cart/deploy.yaml
        '''
    }
}
```

### Execution Flow

1. Jenkins opens deploy.yaml.
2. Finds existing image reference.
3. Replaces it with latest ECR image.
4. Verifies update.

### Example

Before:

```yaml
image: ghcr.io/open-telemetry/demo:1.12.0-cartservice
```

After:

```yaml
image: 004058506543.dkr.ecr.ap-south-1.amazonaws.com/open-telemetry-registry:25
```

### Importance

* Ensures Kubernetes deploys the latest build.
* Eliminates manual YAML modifications.

---

## Stage 8: Deploy To Kubernetes

### Objective

Deploy Cart Service into Kubernetes cluster.

### Pipeline Code

```groovy
stage('Deploy To Kubernetes') {
    steps {
        sh '''
        kubectl apply -f src/cart/kubernetes/cart/deploy.yaml
        kubectl apply -f src/cart/kubernetes/cart/svc.yaml
        '''
    }
}
```

### Resources Applied

| Resource   | Purpose                 |
| ---------- | ----------------------- |
| Deployment | Pod Management          |
| Service    | Internal Cluster Access |

### Execution Flow

1. Deployment manifest applied.
2. Service manifest applied.
3. Kubernetes detects image update.
4. Rolling deployment starts.

---

## Stage 9: Rollout Status Verification

### Objective

Verify successful deployment rollout.

### Pipeline Code

```groovy
stage('Rollout Status') {
    steps {
        sh '''
        kubectl rollout status deployment/${K8S_DEPLOYMENT} -n ${NAMESPACE}
        '''
    }
}
```

### Execution Flow

1. Kubernetes creates new pods.
2. Readiness checks pass.
3. Old pods terminate.
4. Deployment completes successfully.

### Expected Output

```text
deployment "opentelemetry-demo-cartservice" successfully rolled out
```

---

# 6. Post Actions

## Success

```groovy
success {
    echo 'Cart Service Deployment Successful'
}
```

Displays successful deployment message.

## Failure

```groovy
failure {
    echo 'Pipeline Failed'
}
```

Displays failure notification.

## Cleanup

```groovy
always {
    sh '''
    docker image prune -f || true
    '''
}
```

Removes unused Docker images and frees Jenkins server disk space.

---

# 7. Outcome

After successful execution of the pipeline:

* Cart Service source code is retrieved from GitHub.
* Docker image is built automatically.
* Image is pushed to Amazon ECR.
* Kubernetes manifests are updated.
* Latest version is deployed to the Kubernetes cluster.
* Rollout status is verified.
* Temporary Docker artifacts are cleaned up automatically.

This implementation provides a fully automated CI/CD workflow for Cart Service deployment using Jenkins, Docker, Amazon ECR, and Kubernetes.
