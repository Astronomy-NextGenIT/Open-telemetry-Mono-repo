# Accounting Service CI/CD Pipeline Documentation

## Overview

This Jenkins pipeline automates the build, packaging, deployment, and verification process for the Accounting Service in the OpenTelemetry Demo application.

The pipeline performs the following operations:

1. Source Code Checkout from GitHub
2. Tool Verification
3. Authentication to AWS Public ECR
4. Docker Image Build
5. Docker Image Tagging
6. Docker Image Push to AWS Public ECR
7. Kubernetes Manifest Update
8. Deployment to Kubernetes Cluster
9. Deployment Rollout Verification
10. Cleanup of Local Docker Images

---

# Pipeline Configuration

## Environment Variables

| Variable       | Description                                              |
| -------------- | -------------------------------------------------------- |
| AWS_REGION     | AWS Region used for Public ECR authentication            |
| PUBLIC_ECR_URI | Public ECR repository URI                                |
| IMAGE_NAME     | Docker image name for Accounting Service                 |
| IMAGE_TAG      | Versioned image tag generated using Jenkins build number |
| K8S_DEPLOYMENT | Kubernetes deployment name                               |
| NAMESPACE      | Kubernetes namespace                                     |
| TARGETARCH     | Target architecture for .NET build                       |

```groovy
environment {

    AWS_REGION = "us-east-1"

    PUBLIC_ECR_URI = "public.ecr.aws/q2a1e2p7/open-telemetry-demo"

    IMAGE_NAME = "accountingservice"
    IMAGE_TAG = "${IMAGE_NAME}-${BUILD_NUMBER}"

    K8S_DEPLOYMENT = "opentelemetry-demo-accountingservice"
    NAMESPACE = "default"

    TARGETARCH = "amd64"
}
```

---

# Stage 1: Checkout

## Purpose

Fetch the latest Accounting Service source code from GitHub.

## Actions

* Connect to GitHub repository
* Checkout the main branch

```groovy
stage('Checkout') {
    steps {
        git branch: 'main',
            url: 'https://github.com/Astronomy-NextGenIT/Open-telemetry-Mono-repo.git'
    }
}
```

---

# Stage 2: Verify Tools

## Purpose

Verify required tools are available on Jenkins Agent.

## Tools Checked

* AWS CLI
* Kubernetes CLI (kubectl)
* Docker

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

---

# Stage 3: Login to AWS Public ECR

## Purpose

Authenticate Docker with AWS Public Elastic Container Registry.

## Actions

* Generate ECR login token
* Authenticate Docker client

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

---

# Stage 4: Build Docker Image

## Purpose

Build Accounting Service Docker image.

## Build Details

* Uses Dockerfile located at:

```text
src/accounting/Dockerfile
```

* Builds image for Linux AMD64 architecture
* Passes TARGETARCH build argument

```groovy
stage('Build Docker Image') {
    steps {
        sh '''
        docker build \
        --platform linux/amd64 \
        --build-arg TARGETARCH=${TARGETARCH} \
        -t ${IMAGE_NAME}:${IMAGE_TAG} \
        -f src/accounting/Dockerfile .
        '''
    }
}
```

## Example Generated Image

```text
accountingservice:accountingservice-12
```

---

# Stage 5: Tag Docker Image

## Purpose

Create tags for ECR deployment.

## Tags Created

### Build Specific Tag

```text
public.ecr.aws/q2a1e2p7/open-telemetry-demo:accountingservice-12
```

### Latest Tag

```text
public.ecr.aws/q2a1e2p7/open-telemetry-demo:accountingservice-latest
```

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

---

# Stage 6: Push Image to Public ECR

## Purpose

Upload Docker images to AWS Public ECR.

## Actions

* Push build-specific image
* Push latest image

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

---

# Stage 7: Update Kubernetes Manifest

## Purpose

Update deployment manifest with newly built image.

## Actions

* Replace image reference in deployment file
* Verify updated image

Deployment file:

```text
src/accounting/kubernetes/accounting/deploy.yaml
```

```groovy
stage('Update Manifest') {
    steps {
        sh '''
        sed -i "s|image:.*|image: ${PUBLIC_ECR_URI}:${IMAGE_TAG}|g" \
        src/accounting/kubernetes/accounting/deploy.yaml

        echo "Updated Image:"
        grep -n "image:" \
        src/accounting/kubernetes/accounting/deploy.yaml
        '''
    }
}
```

---

# Stage 8: Deploy to Kubernetes

## Purpose

Deploy updated Accounting Service image to Kubernetes cluster.

## Actions

Apply deployment manifest:

```groovy
stage('Deploy To Kubernetes') {
    steps {
        sh '''
        kubectl apply -f src/accounting/kubernetes/accounting/deploy.yaml
        '''
    }
}
```

---

# Stage 9: Rollout Verification

## Purpose

Ensure deployment completed successfully.

## Actions

* Monitor deployment rollout
* Wait up to 300 seconds

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

---

# Post Build Actions

## Success

Displays deployment success message.

```groovy
success {
    echo "Accounting Service Deployment Successful"
    echo "Image Pushed: ${PUBLIC_ECR_URI}:${IMAGE_TAG}"
}
```

---

## Failure

Displays deployment failure message.

```groovy
failure {
    echo "Accounting Service Pipeline Failed"
}
```

---

## Always

Clean up unused Docker images from Jenkins agent.

```groovy
always {
    sh '''
    docker image prune -af || true
    '''
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

        IMAGE_NAME = "accountingservice"
        IMAGE_TAG = "${IMAGE_NAME}-${BUILD_NUMBER}"

        K8S_DEPLOYMENT = "opentelemetry-demo-accountingservice"
        NAMESPACE = "default"

        TARGETARCH = "amd64"
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
                --platform linux/amd64 \
                --build-arg TARGETARCH=${TARGETARCH} \
                -t ${IMAGE_NAME}:${IMAGE_TAG} \
                -f src/accounting/Dockerfile .
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
                src/accounting/kubernetes/accounting/deploy.yaml

                echo "Updated Image:"
                grep -n "image:" \
                src/accounting/kubernetes/accounting/deploy.yaml
                '''
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                sh '''
                kubectl apply -f src/accounting/kubernetes/accounting/deploy.yaml
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
            echo "Accounting Service Deployment Successful"
            echo "Image Pushed: ${PUBLIC_ECR_URI}:${IMAGE_TAG}"
        }

        failure {
            echo "Accounting Service Pipeline Failed"
        }

        always {
            sh '''
            docker image prune -af || true
            '''
        }
    }
}
```

## Outcome

After successful execution:

* Docker image is built.
* Image is pushed to AWS Public ECR.
* Kubernetes deployment is updated.
* New pod is rolled out successfully.
* Jenkins workspace remains clean through image pruning.
