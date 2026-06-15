# Recommendation Service CI/CD Pipeline Documentation

## Overview

This Jenkins pipeline automates the complete CI/CD process for the Recommendation Service of the OpenTelemetry Demo application.

The pipeline performs the following tasks:

1. Clones the latest source code from GitHub.
2. Verifies required tools are available.
3. Authenticates with AWS Public ECR.
4. Builds the Recommendation Service Docker image.
5. Tags the image with both a build-specific tag and a latest tag.
6. Pushes the image to AWS Public ECR.
7. Updates the Kubernetes deployment manifest with the new image version.
8. Deploys the application to Kubernetes.
9. Verifies successful rollout.
10. Cleans up local Docker images.

---

## Repository Structure

```text
Open-telemetry-Mono-repo/
│
├── src/
│   └── recommendation/
│       ├── Dockerfile
│       ├── kubernetes/
│       │   └── recommendation/
│       │       ├── deploy.yaml
│       │       └── svc.yaml
│       └── source files
│
└── Jenkinsfile
```

---

# Jenkins Pipeline

```groovy
pipeline {
    agent any

    environment {

        AWS_REGION = "us-east-1"

        PUBLIC_ECR_URI = "public.ecr.aws/q2a1e2p7/open-telemetry-demo"

        IMAGE_NAME = "recommendationservice"
        IMAGE_TAG = "${IMAGE_NAME}-${BUILD_NUMBER}"

        K8S_DEPLOYMENT = "opentelemetry-demo-recommendationservice"
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
                -f src/recommendation/Dockerfile .
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
                src/recommendation/kubernetes/recommendation/deploy.yaml

                echo "Updated Image:"
                grep -n "image:" \
                src/recommendation/kubernetes/recommendation/deploy.yaml
                '''
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                sh '''
                kubectl apply -f src/recommendation/kubernetes/recommendation/deploy.yaml
                kubectl apply -f src/recommendation/kubernetes/recommendation/svc.yaml
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
            echo "RecommendationService Deployment Successful"
            echo "Image Pushed: ${PUBLIC_ECR_URI}:${IMAGE_TAG}"
        }

        failure {
            echo "RecommendationService Pipeline Failed"
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

# Pipeline Stages

## 1. Checkout

Downloads the latest source code from the GitHub repository.

Purpose:

* Fetch application source code
* Fetch Dockerfile
* Fetch Kubernetes manifests

---

## 2. Verify Tools

Verifies required tools are installed and accessible.

Commands executed:

```bash
aws --version
kubectl version --client
docker --version
```

---

## 3. Login Public ECR

Authenticates Docker with AWS Public ECR.

Command:

```bash
aws ecr-public get-login-password \
--region us-east-1 | \
docker login \
--username AWS \
--password-stdin public.ecr.aws
```

---

## 4. Build Docker Image

Builds the Recommendation Service Docker image.

Command:

```bash
docker build \
-t recommendationservice:<build-number> \
-f src/recommendation/Dockerfile .
```

---

## 5. Tag Image

Creates two tags:

Build-specific tag:

```text
recommendationservice-BUILD_NUMBER
```

Latest tag:

```text
recommendationservice-latest
```

---

## 6. Push Image to Public ECR

Uploads Docker images to AWS Public ECR.

Examples:

```text
public.ecr.aws/q2a1e2p7/open-telemetry-demo:recommendationservice-12

public.ecr.aws/q2a1e2p7/open-telemetry-demo:recommendationservice-latest
```

---

## 7. Update Manifest

Updates deploy.yaml with the newly built image.

Example:

Before:

```yaml
image: public.ecr.aws/q2a1e2p7/open-telemetry-demo:recommendationservice-latest
```

After:

```yaml
image: public.ecr.aws/q2a1e2p7/open-telemetry-demo:recommendationservice-12
```

---

## 8. Deploy to Kubernetes

Applies deployment and service manifests.

Commands:

```bash
kubectl apply -f deploy.yaml
kubectl apply -f svc.yaml
```

---

## 9. Rollout Status

Waits until deployment completes successfully.

Command:

```bash
kubectl rollout status deployment/opentelemetry-demo-recommendationservice
```

Timeout:

```text
300 seconds
```

---

## 10. Cleanup

Removes unused Docker images to free disk space.

Command:

```bash
docker image prune -af
```

---

# Verification Commands

Check Pods:

```bash
kubectl get pods
```

Check Deployment:

```bash
kubectl get deployment opentelemetry-demo-recommendationservice
```

Check Service:

```bash
kubectl get svc
```

Check Logs:

```bash
kubectl logs deployment/opentelemetry-demo-recommendationservice
```

Describe Pod:

```bash
kubectl describe pod <pod-name>
```

---

# Troubleshooting

## ImagePullBackOff

Verify:

```bash
docker push
```

completed successfully and image exists in ECR.

---

## CrashLoopBackOff

Inspect logs:

```bash
kubectl logs <pod-name>
```

---

## Rollout Failure

Check:

```bash
kubectl describe deployment opentelemetry-demo-recommendationservice
```

---

## Environment Variable Issues

Verify required variables are configured in deploy.yaml.

Example:

```yaml
- name: PRODUCT_CATALOG_ADDR
  value: opentelemetry-demo-productcatalogservice:8080
```

---

# Expected Outcome

After successful execution:

* Docker image is built.
* Image is pushed to AWS Public ECR.
* Kubernetes deployment is updated.
* Recommendation Service pod is running successfully.
* Application is available within the cluster.
* Jenkins build status shows SUCCESS.
