# CI/CD Pipeline Documentation for Email Service

## Overview

This document describes the Continuous Integration and Continuous Deployment (CI/CD) pipeline implemented for the Email Service application. The pipeline is executed through Jenkins and automates the complete software delivery lifecycle, including source code retrieval, Docker image creation, image publishing to Amazon ECR Public, Kubernetes deployment, and rollout verification.

The objective of this pipeline is to provide a reliable, repeatable, and automated deployment process while maintaining version control of application images through build-specific tagging.

---

# Complete Jenkins Pipeline

```groovy
pipeline {
    agent any

    environment {

        AWS_REGION = "us-east-1"

        PUBLIC_ECR_URI = "public.ecr.aws/q2a1e2p7/open-telemetry-demo"

        IMAGE_NAME = "emailservice"
        IMAGE_TAG = "${IMAGE_NAME}-${BUILD_NUMBER}"

        K8S_DEPLOYMENT = "opentelemetry-demo-emailservice"
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
                -f src/email/Dockerfile .
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
                src/email/kubernetes/email/deploy.yaml

                echo "Updated Image:"
                grep -n "image:" src/email/kubernetes/email/deploy.yaml
                '''
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                sh '''
                kubectl apply -f src/email/kubernetes/email/deploy.yaml
                kubectl apply -f src/email/kubernetes/email/svc.yaml
                '''
            }
        }

        stage('Rollout Status') {
            steps {
                sh '''
                kubectl rollout status deployment/${K8S_DEPLOYMENT} \
                -n ${NAMESPACE} \
                --timeout=300s
                '''
            }
        }
    }

    post {

        success {
            echo "Email Service Deployment Successful"
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

---

# Pipeline Configuration

## Environment Variables

The pipeline uses the following environment variables to maintain flexibility and avoid hardcoded values.

| Variable       | Description                                           |
| -------------- | ----------------------------------------------------- |
| AWS_REGION     | AWS Region used for Amazon ECR Public authentication  |
| PUBLIC_ECR_URI | Amazon ECR Public repository URI                      |
| IMAGE_NAME     | Docker image name                                     |
| IMAGE_TAG      | Unique image tag generated using Jenkins build number |
| K8S_DEPLOYMENT | Kubernetes deployment name                            |
| NAMESPACE      | Kubernetes namespace                                  |

### Configured Values

```text
AWS_REGION=us-east-1

PUBLIC_ECR_URI=public.ecr.aws/q2a1e2p7/open-telemetry-demo

IMAGE_NAME=emailservice

IMAGE_TAG=emailservice-${BUILD_NUMBER}

K8S_DEPLOYMENT=opentelemetry-demo-emailservice

NAMESPACE=default
```

---

# Pipeline Stages

## Stage 1: Checkout Source Code

### Purpose

Retrieves the latest Email Service source code from the GitHub repository.

### Activities

* Connects to GitHub repository.
* Accesses the main branch.
* Downloads source code into Jenkins workspace.

### Repository

```text
https://github.com/Astronomy-NextGenIT/Open-telemetry-Mono-repo.git
```

### Outcome

Latest application code becomes available for the build process.

---

## Stage 2: Verify Required Tools

### Purpose

Validates that all required tools are installed and available on the Jenkins agent before execution.

### Tools Verified

* AWS CLI
* Kubernetes CLI (kubectl)
* Docker Engine

### Commands

```bash
aws --version

kubectl version --client

docker --version
```

### Outcome

Ensures build execution environment is properly configured.

---

## Stage 3: Login to Amazon ECR Public

### Purpose

Authenticates Docker with Amazon ECR Public Registry.

### Activities

* Generates a temporary login token.
* Authenticates Docker using AWS credentials.
* Establishes access to Amazon ECR Public repository.

### Command

```bash
aws ecr-public get-login-password \
--region us-east-1 | \
docker login \
--username AWS \
--password-stdin public.ecr.aws
```

### Outcome

Docker can push images to Amazon ECR Public.

---

## Stage 4: Build Docker Image

### Purpose

Creates a container image for the Email Service application.

### Dockerfile Location

```text
src/email/Dockerfile
```

### Build Command

```bash
docker build \
-t emailservice:emailservice-${BUILD_NUMBER} \
-f src/email/Dockerfile .
```

### Activities

* Reads Dockerfile instructions.
* Builds application binaries.
* Packages application into a Docker image.
* Assigns build-specific tag.

### Outcome

A Docker image is generated locally on the Jenkins agent.

---

## Stage 5: Tag Docker Image

### Purpose

Creates repository tags required for image publication.

### Generated Tags

#### Versioned Image

```text
public.ecr.aws/q2a1e2p7/open-telemetry-demo:emailservice-BUILD_NUMBER
```

#### Latest Image

```text
public.ecr.aws/q2a1e2p7/open-telemetry-demo:emailservice-latest
```

### Commands

```bash
docker tag \
emailservice:emailservice-${BUILD_NUMBER} \
public.ecr.aws/q2a1e2p7/open-telemetry-demo:emailservice-${BUILD_NUMBER}

docker tag \
emailservice:emailservice-${BUILD_NUMBER} \
public.ecr.aws/q2a1e2p7/open-telemetry-demo:emailservice-latest
```

### Outcome

Image becomes ready for publication to Amazon ECR Public.

---

## Stage 6: Push Image to Amazon ECR Public

### Purpose

Uploads container images to the public registry.

### Commands

```bash
docker push public.ecr.aws/q2a1e2p7/open-telemetry-demo:emailservice-${BUILD_NUMBER}

docker push public.ecr.aws/q2a1e2p7/open-telemetry-demo:emailservice-latest
```

### Activities

* Pushes build-specific image.
* Pushes latest image.

### Outcome

Container images become available for deployment.

---

## Stage 7: Update Kubernetes Deployment Manifest

### Purpose

Updates Kubernetes deployment configuration with the newly generated image tag.

### Deployment Manifest

```text
src/email/kubernetes/email/deploy.yaml
```

### Command

```bash
sed -i "s|image:.*|image: public.ecr.aws/q2a1e2p7/open-telemetry-demo:emailservice-${BUILD_NUMBER}|g" \
src/email/kubernetes/email/deploy.yaml
```

### Verification Command

```bash
grep -n "image:" src/email/kubernetes/email/deploy.yaml
```

### Outcome

Deployment manifest references the latest image created during the current build.

---

## Stage 8: Deploy to Kubernetes

### Purpose

Deploys Email Service resources to the Kubernetes cluster.

### Resources Applied

#### Deployment

```text
src/email/kubernetes/email/deploy.yaml
```

#### Service

```text
src/email/kubernetes/email/svc.yaml
```

### Commands

```bash
kubectl apply -f src/email/kubernetes/email/deploy.yaml

kubectl apply -f src/email/kubernetes/email/svc.yaml
```

### Outcome

Kubernetes updates the Email Service deployment with the newly published image.

---

## Stage 9: Verify Rollout Status

### Purpose

Validates that Kubernetes successfully completes the deployment.

### Command

```bash
kubectl rollout status \
deployment/opentelemetry-demo-emailservice \
-n default \
--timeout=300s
```

### Activities

* Monitors deployment progress.
* Waits for pods to become ready.
* Confirms successful rollout.

### Outcome

Deployment status is validated before pipeline completion.

---

# Post Build Actions

## Success Action

Executed when all pipeline stages complete successfully.

### Output

```text
Email Service Deployment Successful

Image Pushed:
public.ecr.aws/q2a1e2p7/open-telemetry-demo:emailservice-BUILD_NUMBER
```

### Purpose

Provides deployment confirmation and image information.

---

## Failure Action

Executed if any stage fails.

### Output

```text
Pipeline Failed
```

### Purpose

Indicates deployment failure for troubleshooting.

---

## Cleanup Action

Executed regardless of pipeline result.

### Command

```bash
docker image prune -af || true
```

### Purpose

* Removes unused Docker images.
* Frees storage space.
* Prevents disk exhaustion on Jenkins agents.

---

# Benefits of the Pipeline

## Automated CI/CD Process

Eliminates manual build and deployment activities.

## Image Versioning

Every build produces a uniquely tagged container image.

## Latest Image Availability

Maintains a dedicated latest tag for quick access to the most recent version.

## Faster Deployments

Automates image publishing and Kubernetes deployment.

## Deployment Validation

Automatically verifies successful rollout before marking the pipeline as successful.

## Consistency

Ensures repeatable and reliable deployments across environments.

---

# End-to-End Workflow

```text
GitHub Repository
        │
        ▼
Checkout Source Code
        │
        ▼
Verify Tools
        │
        ▼
Login to Amazon ECR Public
        │
        ▼
Build Docker Image
        │
        ▼
Tag Docker Image
        │
        ▼
Push Image to Amazon ECR Public
        │
        ▼
Update Kubernetes Manifest
        │
        ▼
Deploy to Kubernetes
        │
        ▼
Verify Rollout Status
        │
        ▼
Email Service Successfully Running
```
