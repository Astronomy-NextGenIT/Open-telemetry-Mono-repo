
# Checkout Service CI/CD Pipeline Documentation

## Overview

This Jenkins pipeline automates the build, containerization, deployment, and rollout verification of the Checkout Service application. The pipeline performs the following tasks:

* Clones source code from GitHub.
* Verifies required tools.
* Authenticates with Amazon ECR.
* Builds a Docker image.
* Pushes the image to Amazon ECR.
* Updates Kubernetes deployment manifests.
* Deploys the application to Kubernetes.
* Verifies deployment rollout status.
* Cleans up unused Docker images.

---

## Pipeline Configuration

### Environment Variables

| Variable       | Description                            |
| -------------- | -------------------------------------- |
| AWS_REGION     | AWS region where ECR repository exists |
| AWS_ACCOUNT_ID | AWS Account ID                         |
| ECR_REPO       | ECR repository name                    |
| IMAGE_NAME     | Docker image name (checkout)           |
| IMAGE_TAG      | Jenkins build number used as image tag |
| ECR_URI        | Complete ECR repository URI            |
| K8S_DEPLOYMENT | Kubernetes deployment name             |
| NAMESPACE      | Kubernetes namespace                   |

```groovy
environment {
    AWS_REGION = "ap-south-1"
    AWS_ACCOUNT_ID = "004058506543"
    ECR_REPO = "open-telemetry-registry"
    IMAGE_NAME = "checkout"
    IMAGE_TAG = "${BUILD_NUMBER}"
    ECR_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}"
    K8S_DEPLOYMENT = "opentelemetry-demo-checkoutservice"
    NAMESPACE = "default"
}
```

---

## Pipeline Stages

### Stage 1: Checkout

#### Purpose

Clones the latest application source code from the GitHub repository.

#### Actions Performed

* Connects to GitHub.
* Checks out the `main` branch.
* Downloads the latest source code into the Jenkins workspace.

```groovy
stage('Checkout') {
    steps {
        git branch: 'main',
        url: 'https://github.com/Astronomy-NextGenIT/Open-telemetry-Mono-repo.git'
    }
}
```

#### Output

Latest Checkout Service source code becomes available in Jenkins workspace.

---

### Stage 2: Verify Tools

#### Purpose

Ensures that all required tools are installed on the Jenkins server.

#### Actions Performed

Checks versions of:

* AWS CLI
* Kubectl
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

#### Output

Displays installed tool versions in Jenkins console logs.

---

### Stage 3: ECR Login

#### Purpose

Authenticates Docker with Amazon Elastic Container Registry (ECR).

#### Actions Performed

* Retrieves ECR authentication token.
* Logs Docker into ECR.

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

#### Output

Docker is successfully authenticated with Amazon ECR.

---

### Stage 4: Build Docker Image

#### Purpose

Builds a Docker image for the Checkout Service.

#### Actions Performed

Uses the Checkout Service Dockerfile:

```text
src/checkout/Dockerfile
```

Build command:

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

#### Example

```text
checkout:25
```

Where:

* checkout = image name
* 25 = Jenkins build number

#### Output

A Docker image is created locally on the Jenkins server.

---

### Stage 5: Tag Image

#### Purpose

Tags the locally built Docker image with the ECR repository URI.

#### Actions Performed

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

#### Example

```text
004058506543.dkr.ecr.ap-south-1.amazonaws.com/open-telemetry-registry:25
```

#### Output

Docker image is prepared for upload to ECR.

---

### Stage 6: Push Image To ECR

#### Purpose

Uploads the Docker image to Amazon ECR.

#### Actions Performed

```groovy
stage('Push Image To ECR') {
    steps {
        sh '''
        docker push ${ECR_URI}:${IMAGE_TAG}
        '''
    }
}
```

#### Output

Docker image is stored in Amazon ECR and available for deployment.

---

### Stage 7: Update Manifest

#### Purpose

Updates the Kubernetes deployment manifest with the newly created image.

#### Actions Performed

Replaces the default image:

```text
ghcr.io/open-telemetry/demo:1.12.0-checkoutservice
```

With:

```text
004058506543.dkr.ecr.ap-south-1.amazonaws.com/open-telemetry-registry:<BUILD_NUMBER>
```

Command:

```groovy
stage('Update Manifest') {
    steps {
        sh '''
        sed -i "s|ghcr.io/open-telemetry/demo:1.12.0-checkoutservice|${ECR_URI}:${IMAGE_TAG}|g" \
        src/checkout/kubernetes/checkout/deploy.yaml

        echo "Updated Image:"
        grep -n "image:" src/checkout/kubernetes/checkout/deploy.yaml
        '''
    }
}
```

#### Output

Deployment manifest is updated with the latest image version.

---

### Stage 8: Deploy To Kubernetes

#### Purpose

Deploys the Checkout Service to the Kubernetes cluster.

#### Actions Performed

Applies:

1. Deployment Manifest
2. Service Manifest

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

#### Files Used

```text
src/checkout/kubernetes/checkout/deploy.yaml
src/checkout/kubernetes/checkout/svc.yaml
```

#### Output

Kubernetes resources are created or updated.

---

### Stage 9: Rollout Status

#### Purpose

Verifies that the Kubernetes deployment completes successfully.

#### Actions Performed

```groovy
stage('Rollout Status') {
    steps {
        sh '''
        kubectl rollout status deployment/${K8S_DEPLOYMENT} -n ${NAMESPACE}
        '''
    }
}
```

#### Deployment Monitored

```text
opentelemetry-demo-checkoutservice
```

#### Output

Jenkins waits until:

```text
deployment "opentelemetry-demo-checkoutservice" successfully rolled out
```

---

## Post Actions

### Success

Executed when the pipeline completes successfully.

```groovy
success {
    echo 'Checkout Service Deployment Successful'
}
```

Output:

```text
Checkout Service Deployment Successful
```

---

### Failure

Executed if any stage fails.

```groovy
failure {
    echo 'Pipeline Failed'
}
```

Output:

```text
Pipeline Failed
```

---

### Always

Runs regardless of pipeline result.

#### Purpose

Removes unused Docker images from the Jenkins server to save disk space.

```groovy
always {
    sh '''
    docker image prune -f || true
    '''
}
```

#### Benefits

* Frees disk space.
* Prevents Jenkins server storage issues.
* Keeps Docker environment clean.

---

# Complete CI/CD Flow

```text
Developer Pushes Code
          │
          ▼
GitHub Repository
          │
          ▼
Jenkins Checkout Stage
          │
          ▼
Verify Tools
          │
          ▼
ECR Login
          │
          ▼
Build Docker Image
          │
          ▼
Tag Image
          │
          ▼
Push Image To ECR
          │
          ▼
Update Kubernetes Manifest
          │
          ▼
Deploy To Kubernetes
          │
          ▼
Check Rollout Status
          │
          ▼
Checkout Service Available in Kubernetes Cluster
```

## Outcome

This pipeline provides a complete CI/CD workflow for the Checkout Service by automatically:

* Building Docker images.
* Storing images in Amazon ECR.
* Updating Kubernetes manifests.
* Deploying to Kubernetes.
* Verifying successful rollout.
* Cleaning up unused Docker images after execution.

This ensures fast, consistent, and reliable deployments of the Checkout Service application.
