# LoadGenerator CI/CD Pipeline Documentation

## Overview

This Jenkins pipeline automates the complete CI/CD workflow for the **LoadGenerator** service in the OpenTelemetry Demo application.

The pipeline performs the following tasks:

1. Clones the source code from GitHub.
2. Verifies required tools are available.
3. Authenticates with AWS Public ECR.
4. Builds the LoadGenerator Docker image.
5. Tags the image with build-specific and latest tags.
6. Pushes the image to Public ECR.
7. Updates the Kubernetes deployment manifest with the new image.
8. Deploys the updated resources to Kubernetes.
9. Verifies successful rollout of the deployment.
10. Cleans up unused Docker images after execution.

---

# Pipeline Configuration

```groovy
environment {

    AWS_REGION = "us-east-1"

    PUBLIC_ECR_URI = "public.ecr.aws/q2a1e2p7/open-telemetry-demo"

    IMAGE_NAME = "loadgenerator"
    IMAGE_TAG = "${IMAGE_NAME}-${BUILD_NUMBER}"

    K8S_DEPLOYMENT = "opentelemetry-demo-loadgenerator"
    NAMESPACE = "default"
}
```

### Environment Variables

| Variable       | Description                                   |
| -------------- | --------------------------------------------- |
| AWS_REGION     | AWS region used for Public ECR authentication |
| PUBLIC_ECR_URI | Public ECR repository URI                     |
| IMAGE_NAME     | Docker image name                             |
| IMAGE_TAG      | Build-specific image tag                      |
| K8S_DEPLOYMENT | Kubernetes deployment name                    |
| NAMESPACE      | Kubernetes namespace                          |

---

# Pipeline Stages

## Stage 1: Checkout

### Purpose

Retrieves the latest application source code from the GitHub repository.

### Implementation

```groovy
stage('Checkout') {
    steps {
        git branch: 'main',
            url: 'https://github.com/Astronomy-NextGenIT/Open-telemetry-Mono-repo.git'
    }
}
```

### Outcome

The Jenkins workspace contains the latest version of the repository.

---

## Stage 2: Verify Tools

### Purpose

Validates that all required tools are installed and accessible on the Jenkins agent.

### Implementation

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

### Outcome

Displays versions of:

* AWS CLI
* kubectl
* Docker

---

## Stage 3: Login Public ECR

### Purpose

Authenticates Docker with AWS Public Elastic Container Registry.

### Implementation

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

### Outcome

The Jenkins agent receives authorization to push Docker images to Public ECR.

---

## Stage 4: Build Docker Image

### Purpose

Builds the LoadGenerator Docker image from the repository source code.

### Implementation

```groovy
stage('Build Docker Image') {
    steps {
        sh '''
        docker build \
        -t ${IMAGE_NAME}:${IMAGE_TAG} \
        -f src/load-generator/Dockerfile .
        '''
    }
}
```

### Dockerfile Location

```text
src/load-generator/Dockerfile
```

### Outcome

Creates a Docker image:

```text
loadgenerator:<build-number>
```

Example:

```text
loadgenerator:loadgenerator-15
```

---

## Stage 5: Tag Image

### Purpose

Creates tags for both the build-specific image and the latest image.

### Implementation

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

### Generated Tags

Build-specific:

```text
public.ecr.aws/q2a1e2p7/open-telemetry-demo:loadgenerator-15
```

Latest:

```text
public.ecr.aws/q2a1e2p7/open-telemetry-demo:loadgenerator-latest
```

### Outcome

Both image tags are prepared for publishing.

---

## Stage 6: Push Image To Public ECR

### Purpose

Publishes Docker images to AWS Public ECR.

### Implementation

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

### Outcome

The repository receives:

```text
loadgenerator-<build-number>
loadgenerator-latest
```

tags.

---

## Stage 7: Update Manifest

### Purpose

Updates the Kubernetes deployment manifest to use the newly built image.

### Implementation

```groovy
stage('Update Manifest') {
    steps {
        sh '''
        sed -i "s|image:.*|image: ${PUBLIC_ECR_URI}:${IMAGE_TAG}|g" \
        src/load-generator/kubernetes/loadgenerator/deploy.yaml

        echo "Updated Image:"
        grep -n "image:" \
        src/load-generator/kubernetes/loadgenerator/deploy.yaml
        '''
    }
}
```

### Manifest Location

```text
src/load-generator/kubernetes/loadgenerator/deploy.yaml
```

### Outcome

The deployment manifest references the newly published image tag.

Example:

```yaml
image: public.ecr.aws/q2a1e2p7/open-telemetry-demo:loadgenerator-15
```

---

## Stage 8: Deploy To Kubernetes

### Purpose

Applies the updated Kubernetes manifests to the cluster.

### Implementation

```groovy
stage('Deploy To Kubernetes') {
    steps {
        sh '''
        kubectl apply -f src/load-generator/kubernetes/loadgenerator/deploy.yaml
        kubectl apply -f src/load-generator/kubernetes/loadgenerator/svc.yaml
        '''
    }
}
```

### Applied Resources

#### Deployment

```text
src/load-generator/kubernetes/loadgenerator/deploy.yaml
```

#### Service

```text
src/load-generator/kubernetes/loadgenerator/svc.yaml
```

### Outcome

Kubernetes updates the LoadGenerator deployment and service.

---

## Stage 9: Rollout Status

### Purpose

Monitors deployment progress and verifies successful rollout.

### Implementation

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

### Outcome

Waits until the deployment becomes healthy and available.

Target deployment:

```text
opentelemetry-demo-loadgenerator
```

---

# Post Actions

## Success

### Purpose

Displays deployment success information.

### Implementation

```groovy
success {
    echo "LoadGenerator Deployment Successful"
    echo "Image Pushed: ${PUBLIC_ECR_URI}:${IMAGE_TAG}"
}
```

### Example Output

```text
LoadGenerator Deployment Successful
Image Pushed: public.ecr.aws/q2a1e2p7/open-telemetry-demo:loadgenerator-15
```

---

## Failure

### Purpose

Displays failure notification.

### Implementation

```groovy
failure {
    echo "LoadGenerator Pipeline Failed"
}
```

### Example Output

```text
LoadGenerator Pipeline Failed
```

---

## Cleanup

### Purpose

Removes unused Docker images from the Jenkins agent to reclaim disk space.

### Implementation

```groovy
always {
    sh '''
    docker image prune -af || true
    '''
}
```

### Outcome

Deletes dangling and unused Docker images after every pipeline execution.

---

# Pipeline Flow

```text
Checkout Source Code
        │
        ▼
Verify Required Tools
        │
        ▼
Login to AWS Public ECR
        │
        ▼
Build LoadGenerator Docker Image
        │
        ▼
Tag Docker Image
        │
        ▼
Push Image to Public ECR
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
Cleanup Docker Images
```

---

# Result

Upon successful execution, the pipeline:

* Builds the latest LoadGenerator container image.
* Publishes the image to AWS Public ECR.
* Updates the Kubernetes deployment manifest.
* Deploys the new version to the Kubernetes cluster.
* Verifies deployment health and availability.
* Cleans up temporary Docker artifacts from the Jenkins agent.
