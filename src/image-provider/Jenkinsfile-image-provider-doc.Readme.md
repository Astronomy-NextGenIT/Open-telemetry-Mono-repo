# ImageProvider CI/CD Pipeline Documentation

## Project Overview

The ImageProvider service is part of the OpenTelemetry Demo application. This service serves static image content using an NGINX-based container.

A Jenkins CI/CD pipeline has been implemented to automate:

* Source code checkout from GitHub
* Docker image build
* Docker image tagging
* Push image to AWS Public ECR
* Update Kubernetes deployment manifest
* Deploy manifests to Kubernetes
* Verify successful rollout

---

# Service Information

| Parameter                | Value                                             |
| ------------------------ | ------------------------------------------------- |
| Service Name             | imageprovider                                     |
| Kubernetes Deployment    | opentelemetry-demo-imageprovider                  |
| Namespace                | default                                           |
| Dockerfile Location      | src/image-provider/Dockerfile                     |
| Deployment Manifest      | src/image-provider/kubernetes/deploy.yaml         |
| Service Manifest         | src/image-provider/kubernetes/svc.yaml            |
| Service Account Manifest | src/image-provider/kubernetes/serviceaccount.yaml |
| Public ECR Repository    | public.ecr.aws/q2a1e2p7/open-telemetry-demo       |

---

# CI/CD Workflow

GitHub Repository

↓

Jenkins Pipeline

↓

Docker Build

↓

AWS Public ECR

↓

Manifest Update

↓

Kubernetes Deployment

↓

Rollout Verification

---

# Jenkins Pipeline

```groovy
pipeline {
    agent any

    environment {

        AWS_REGION = "us-east-1"

        PUBLIC_ECR_URI = "public.ecr.aws/q2a1e2p7/open-telemetry-demo"

        IMAGE_NAME = "imageprovider"
        IMAGE_TAG = "${IMAGE_NAME}-${BUILD_NUMBER}"

        K8S_DEPLOYMENT = "opentelemetry-demo-imageprovider"
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
                -f src/image-provider/Dockerfile .
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
                src/image-provider/kubernetes/deploy.yaml

                echo "Updated Image:"
                grep -n "image:" \
                src/image-provider/kubernetes/deploy.yaml
                '''
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                sh '''
                kubectl apply -f src/image-provider/kubernetes/serviceaccount.yaml
                kubectl apply -f src/image-provider/kubernetes/deploy.yaml
                kubectl apply -f src/image-provider/kubernetes/svc.yaml
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
            echo "ImageProvider Deployment Successful"
            echo "Image Pushed: ${PUBLIC_ECR_URI}:${IMAGE_TAG}"
        }

        failure {
            echo "ImageProvider Pipeline Failed"
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

# Pipeline Stage Explanation

## Stage 1 – Checkout

Purpose:

Download the latest source code from GitHub.

Output:

Latest ImageProvider source code is available in Jenkins workspace.

---

## Stage 2 – Verify Tools

Purpose:

Verify required tools are installed and accessible.

Commands Executed:

```bash
aws --version
kubectl version --client
docker --version
```

---

## Stage 3 – Login Public ECR

Purpose:

Authenticate Docker with AWS Public ECR.

Command:

```bash
aws ecr-public get-login-password \
--region us-east-1 | \
docker login \
--username AWS \
--password-stdin public.ecr.aws
```

Expected Output:

```text
Login Succeeded
```

---

## Stage 4 – Build Docker Image

Purpose:

Build the ImageProvider Docker image.

Command:

```bash
docker build \
-t imageprovider:imageprovider-${BUILD_NUMBER} \
-f src/image-provider/Dockerfile .
```

Example:

```text
imageprovider:imageprovider-25
```

---

## Stage 5 – Tag Image

Purpose:

Create ECR-compatible tags.

Generated Tags:

```text
public.ecr.aws/q2a1e2p7/open-telemetry-demo:imageprovider-25
public.ecr.aws/q2a1e2p7/open-telemetry-demo:imageprovider-latest
```

---

## Stage 6 – Push Image To Public ECR

Purpose:

Upload image to AWS Public ECR.

Commands:

```bash
docker push public.ecr.aws/q2a1e2p7/open-telemetry-demo:imageprovider-25
docker push public.ecr.aws/q2a1e2p7/open-telemetry-demo:imageprovider-latest
```

---

## Stage 7 – Update Manifest

Purpose:

Replace the existing image in deploy.yaml with the newly built image.

Before:

```yaml
image: ghcr.io/open-telemetry/demo:1.12.0-imageprovider
```

After:

```yaml
image: public.ecr.aws/q2a1e2p7/open-telemetry-demo:imageprovider-25
```

Command:

```bash
sed -i "s|image:.*|image: ${PUBLIC_ECR_URI}:${IMAGE_TAG}|g"
```

---

## Stage 8 – Deploy To Kubernetes

Purpose:

Deploy updated manifests to Kubernetes.

Commands:

```bash
kubectl apply -f serviceaccount.yaml
kubectl apply -f deploy.yaml
kubectl apply -f svc.yaml
```

Resources Created:

* ServiceAccount
* Deployment
* Service

---

## Stage 9 – Rollout Status

Purpose:

Wait for Kubernetes deployment to become healthy.

Command:

```bash
kubectl rollout status deployment/opentelemetry-demo-imageprovider
```

Expected Result:

```text
deployment "opentelemetry-demo-imageprovider" successfully rolled out
```

---

# Post Build Actions

## Success

Displays:

```text
ImageProvider Deployment Successful
Image Pushed: public.ecr.aws/q2a1e2p7/open-telemetry-demo:imageprovider-25
```

## Failure

Displays:

```text
ImageProvider Pipeline Failed
```

## Cleanup

Removes unused Docker images:

```bash
docker image prune -af
```

---

# Verification Commands

## Verify Deployment

```bash
kubectl get deployment opentelemetry-demo-imageprovider
```

---

## Verify Pods

```bash
kubectl get pods | grep imageprovider
```

Expected:

```text
1/1 Running
```

---

## Verify Image

```bash
kubectl describe deployment opentelemetry-demo-imageprovider | grep Image
```

Expected:

```text
Image: public.ecr.aws/q2a1e2p7/open-telemetry-demo:imageprovider-<BUILD_NUMBER>
```

---

# Expected Outcome

After successful execution:

* Docker image is built successfully.
* Image is pushed to AWS Public ECR.
* Kubernetes deployment is updated.
* New ImageProvider pod is created.
* Rollout completes successfully.
* Service remains available without downtime.
* Latest image version is running in the cluster.
