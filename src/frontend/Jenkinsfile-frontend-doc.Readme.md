# Frontend Service CI/CD Pipeline Documentation

## Overview

This Jenkins Pipeline automates the complete CI/CD lifecycle for the Frontend Service of the OpenTelemetry Demo application.

The pipeline performs the following activities:

1. Clones the source code from GitHub.
2. Verifies required tools are installed.
3. Authenticates with AWS Public ECR.
4. Builds the Frontend Docker image.
5. Tags the Docker image.
6. Pushes the image to AWS Public ECR.
7. Updates the Kubernetes deployment manifest with the new image tag.
8. Deploys the application to Kubernetes.
9. Verifies successful rollout of the deployment.
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
Update Kubernetes Manifest
↓
Deploy to Kubernetes Cluster
↓
Verify Rollout Status

---

# Environment Variables

| Variable       | Description                                   |
| -------------- | --------------------------------------------- |
| AWS_REGION     | AWS region used for Public ECR authentication |
| PUBLIC_ECR_URI | Public ECR repository URI                     |
| IMAGE_NAME     | Frontend Docker image name                    |
| IMAGE_TAG      | Dynamic image tag using Jenkins build number  |
| K8S_DEPLOYMENT | Kubernetes deployment name                    |
| NAMESPACE      | Kubernetes namespace                          |

```groovy
environment {

    AWS_REGION = "us-east-1"

    PUBLIC_ECR_URI = "public.ecr.aws/q2a1e2p7/open-telemetry-demo"

    IMAGE_NAME = "frontend"
    IMAGE_TAG = "${IMAGE_NAME}-${BUILD_NUMBER}"

    K8S_DEPLOYMENT = "opentelemetry-demo-frontend"
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

        IMAGE_NAME = "frontend"
        IMAGE_TAG = "${IMAGE_NAME}-${BUILD_NUMBER}"

        K8S_DEPLOYMENT = "opentelemetry-demo-frontend"
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
                -f src/frontend/Dockerfile .
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
                src/frontend/kubernetes/frontend/deploy.yaml

                echo "Updated Image:"
                grep -n "image:" \
                src/frontend/kubernetes/frontend/deploy.yaml
                '''
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                sh '''
                kubectl apply -f src/frontend/kubernetes/frontend/deploy.yaml
                kubectl apply -f src/frontend/kubernetes/frontend/svc.yaml
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
            echo "Frontend Deployment Successful"
            echo "Image Pushed: ${PUBLIC_ECR_URI}:${IMAGE_TAG}"
        }

        failure {
            echo "Frontend Pipeline Failed"
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

## Stage 1: Checkout Source Code

Purpose:

Clone the latest source code from the GitHub repository.

```bash
git clone https://github.com/Astronomy-NextGenIT/Open-telemetry-Mono-repo.git
```

Outcome:

Latest Frontend source code becomes available in Jenkins workspace.

---

## Stage 2: Verify Tools

Purpose:

Validate that required tools are installed on the Jenkins agent.

Commands Executed:

```bash
aws --version
kubectl version --client
docker --version
```

Outcome:

Confirms availability of:

* AWS CLI
* kubectl
* Docker Engine

---

## Stage 3: Login to AWS Public ECR

Purpose:

Authenticate Jenkins with AWS Public Elastic Container Registry.

Command:

```bash
aws ecr-public get-login-password \
--region us-east-1 | \
docker login \
--username AWS \
--password-stdin public.ecr.aws
```

Outcome:

Jenkins can push Docker images to Public ECR.

---

## Stage 4: Build Docker Image

Purpose:

Create a Docker image for the Frontend application.

Command:

```bash
docker build \
-t frontend:<BUILD_NUMBER> \
-f src/frontend/Dockerfile .
```

Example:

```bash
docker build \
-t frontend:12 \
-f src/frontend/Dockerfile .
```

Outcome:

Creates a Docker image containing the Next.js frontend application.

---

## Stage 5: Tag Docker Image

Purpose:

Create versioned and latest tags for the Docker image.

Commands:

```bash
docker tag frontend:12 \
public.ecr.aws/q2a1e2p7/open-telemetry-demo:frontend-12

docker tag frontend:12 \
public.ecr.aws/q2a1e2p7/open-telemetry-demo:frontend-latest
```

Outcome:

Two tags are generated:

* frontend-<BUILD_NUMBER>
* frontend-latest

---

## Stage 6: Push Image to Public ECR

Purpose:

Upload Docker images to AWS Public ECR.

Commands:

```bash
docker push public.ecr.aws/q2a1e2p7/open-telemetry-demo:frontend-12

docker push public.ecr.aws/q2a1e2p7/open-telemetry-demo:frontend-latest
```

Outcome:

Docker image becomes publicly available and can be pulled by Kubernetes.

---

## Stage 7: Update Kubernetes Manifest

Purpose:

Replace the existing image reference with the newly built image.

Command:

```bash
sed -i "s|image:.*|image: public.ecr.aws/q2a1e2p7/open-telemetry-demo:frontend-12|g" \
src/frontend/kubernetes/frontend/deploy.yaml
```

Verification:

```bash
grep -n "image:" \
src/frontend/kubernetes/frontend/deploy.yaml
```

Outcome:

Deployment manifest points to the latest image version.

---

## Stage 8: Deploy to Kubernetes

Purpose:

Deploy updated Frontend resources.

Commands:

```bash
kubectl apply -f src/frontend/kubernetes/frontend/deploy.yaml

kubectl apply -f src/frontend/kubernetes/frontend/svc.yaml
```

Outcome:

Kubernetes updates deployment and service resources.

---

## Stage 9: Verify Rollout Status

Purpose:

Ensure the deployment successfully rolls out.

Command:

```bash
kubectl rollout status \
deployment/opentelemetry-demo-frontend \
-n default \
--timeout=300s
```

Outcome:

Confirms all frontend pods become healthy and available.

---

# Post Build Actions

## Success

```groovy
success {
    echo "Frontend Deployment Successful"
    echo "Image Pushed: ${PUBLIC_ECR_URI}:${IMAGE_TAG}"
}
```

Displays deployment success message.

---

## Failure

```groovy
failure {
    echo "Frontend Pipeline Failed"
}
```

Displays pipeline failure message.

---

## Cleanup

```groovy
always {
    sh '''
    docker image prune -af || true
    '''
}
```

Removes unused Docker images to free disk space.

---

# Validation Commands

Check Deployment

```bash
kubectl get deployment opentelemetry-demo-frontend
```

Check Pods

```bash
kubectl get pods | grep frontend
```

Check Service

```bash
kubectl get svc | grep frontend
```

Check Rollout History

```bash
kubectl rollout history deployment/opentelemetry-demo-frontend
```

---

# Benefits of this Pipeline

* Fully automated CI/CD workflow.
* Dynamic image versioning using Jenkins build numbers.
* Automatic deployment updates.
* Zero manual image tag modifications.
* Rollout verification before pipeline completion.
* Automatic Docker cleanup.
* Faster and consistent frontend releases.
