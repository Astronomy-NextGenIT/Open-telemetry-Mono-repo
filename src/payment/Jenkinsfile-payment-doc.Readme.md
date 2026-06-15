# PaymentService CI/CD Pipeline Documentation

## Overview

This Jenkins pipeline automates the complete CI/CD workflow for the OpenTelemetry Demo PaymentService application.

The pipeline performs the following actions:

1. Clones the latest source code from GitHub.
2. Verifies required tools are installed.
3. Authenticates with AWS Public ECR.
4. Builds the Docker image for PaymentService.
5. Tags the image with both a build-specific tag and latest tag.
6. Pushes the image to AWS Public ECR.
7. Updates the Kubernetes deployment manifest with the new image.
8. Deploys the application to Kubernetes.
9. Verifies successful rollout of the deployment.
10. Cleans up unused Docker images.

---

# Pipeline Configuration

| Variable       | Description                                    |
| -------------- | ---------------------------------------------- |
| AWS_REGION     | AWS region used for Public ECR authentication  |
| PUBLIC_ECR_URI | Public ECR repository URI                      |
| IMAGE_NAME     | Docker image name                              |
| IMAGE_TAG      | Image tag generated using Jenkins build number |
| K8S_DEPLOYMENT | Kubernetes deployment name                     |
| NAMESPACE      | Kubernetes namespace                           |

Current Configuration:

```groovy
AWS_REGION = "us-east-1"
PUBLIC_ECR_URI = "public.ecr.aws/q2a1e2p7/open-telemetry-demo"

IMAGE_NAME = "paymentservice"
IMAGE_TAG = "${IMAGE_NAME}-${BUILD_NUMBER}"

K8S_DEPLOYMENT = "opentelemetry-demo-paymentservice"
NAMESPACE = "default"
```

---

# Jenkins Pipeline

```groovy
pipeline {
    agent any

    environment {

        AWS_REGION = "us-east-1"

        PUBLIC_ECR_URI = "public.ecr.aws/q2a1e2p7/open-telemetry-demo"

        IMAGE_NAME = "paymentservice"
        IMAGE_TAG = "${IMAGE_NAME}-${BUILD_NUMBER}"

        K8S_DEPLOYMENT = "opentelemetry-demo-paymentservice"
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
                -f src/payment/Dockerfile .
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
                src/payment/kubernetes/payment/deploy.yaml

                echo "Updated Image:"
                grep -n "image:" \
                src/payment/kubernetes/payment/deploy.yaml
                '''
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                sh '''
                kubectl apply -f src/payment/kubernetes/payment/deploy.yaml
                kubectl apply -f src/payment/kubernetes/payment/svc.yaml
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
            echo "PaymentService Deployment Successful"
            echo "Image Pushed: ${PUBLIC_ECR_URI}:${IMAGE_TAG}"
        }

        failure {
            echo "PaymentService Pipeline Failed"
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

## 1. Checkout

Clones the latest code from the GitHub repository.

```bash
git clone https://github.com/Astronomy-NextGenIT/Open-telemetry-Mono-repo.git
```

---

## 2. Verify Tools

Ensures the Jenkins agent has:

* AWS CLI
* kubectl
* Docker

installed and accessible.

---

## 3. Login Public ECR

Authenticates Docker with AWS Public ECR.

```bash
aws ecr-public get-login-password
```

---

## 4. Build Docker Image

Builds the PaymentService image using:

```bash
src/payment/Dockerfile
```

Generated image:

```text
paymentservice:<BUILD_NUMBER>
```

---

## 5. Tag Image

Creates two tags:

```text
paymentservice-<BUILD_NUMBER>
paymentservice-latest
```

Example:

```text
public.ecr.aws/q2a1e2p7/open-telemetry-demo:paymentservice-25
public.ecr.aws/q2a1e2p7/open-telemetry-demo:paymentservice-latest
```

---

## 6. Push Image

Pushes both image tags to AWS Public ECR.

```bash
docker push
```

---

## 7. Update Manifest

Updates the Kubernetes deployment file with the latest image tag.

File:

```text
src/payment/kubernetes/payment/deploy.yaml
```

Example replacement:

```yaml
image: public.ecr.aws/q2a1e2p7/open-telemetry-demo:paymentservice-25
```

---

## 8. Deploy To Kubernetes

Applies Kubernetes manifests.

Deployment:

```text
src/payment/kubernetes/payment/deploy.yaml
```

Service:

```text
src/payment/kubernetes/payment/svc.yaml
```

Commands:

```bash
kubectl apply -f deploy.yaml
kubectl apply -f svc.yaml
```

---

## 9. Rollout Status

Waits for successful deployment rollout.

```bash
kubectl rollout status deployment/opentelemetry-demo-paymentservice
```

Timeout:

```text
300 seconds
```

---

## 10. Cleanup

Removes unused Docker images from the Jenkins agent.

```bash
docker image prune -af
```

---

# Kubernetes Resources Deployed

Deployment:

```text
opentelemetry-demo-paymentservice
```

Service:

```text
paymentservice
```

Namespace:

```text
default
```

---

# Expected Outcome

After a successful pipeline execution:

1. Docker image is built.
2. Image is pushed to AWS Public ECR.
3. Kubernetes deployment is updated.
4. New PaymentService pod is created.
5. Rollout completes successfully.
6. Application becomes available through the Kubernetes Service.

---

# Verification Commands

Check deployment:

```bash
kubectl get deployment opentelemetry-demo-paymentservice
```

Check pods:

```bash
kubectl get pods | grep payment
```

Check service:

```bash
kubectl get svc | grep payment
```

Check logs:

```bash
kubectl logs -f deployment/opentelemetry-demo-paymentservice
```
