pipeline {
    agent any

    environment {
        IMAGE_NAME = "nginx-app"
        TAG = "${BUILD_NUMBER}"
        GIT_EMAIL = "jenkins@ci.com"
        GIT_USER  = "Jenkins CI"
        ARGOCD_SERVER = "argocd.example.com"   // 👈 your ArgoCD API endpoint
        ARGOCD_APP    = "nginx-app"            // 👈 your ArgoCD app name
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                sh 'echo "✅ Code checked out successfully"'
            }
        }

        stage('Validate Project') {
            steps {
                sh '''
                    echo "=== Project Structure ==="
                    ls -la

                    echo "=== Checking Dockerfile ==="
                    [ -f Dockerfile ] || { echo "❌ Dockerfile missing"; exit 1; }

                    echo "=== Checking k8s directory ==="
                    [ -d k8s ] || { echo "❌ k8s directory missing"; exit 1; }
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    echo "=== Building Docker Image ==="
                    echo "Image Tag: ${IMAGE_NAME}:${TAG}"

                    if [ -S /var/run/docker.sock ]; then
                        docker build -t ${IMAGE_NAME}:${TAG} .
                        echo "✅ Docker image built"
                    else
                        echo "⚠️ Docker not available — Simulation mode"
                    fi
                """
            }
        }

        stage('Update Kubernetes Manifests') {
            steps {
                sh """
                    echo "=== Updating Kubernetes YAML ==="

                    if [ -f k8s/nginx-deployment.yaml ]; then
                        sed -i 's|nginx-app:[0-9]*|${IMAGE_NAME}:${TAG}|g' k8s/nginx-deployment.yaml
                        echo "✅ Updated image to ${IMAGE_NAME}:${TAG}"
                        cat k8s/nginx-deployment.yaml
                    else
                        echo "❌ k8s/nginx-deployment.yaml not found"
                        exit 1
                    fi
                """
            }
        }

        stage('Commit & Push Updated Manifests') {
            steps {
                sh """
                    echo "=== Committing Updated YAML ==="

                    git config user.email "${GIT_EMAIL}"
                    git config user.name "${GIT_USER}"

                    git add k8s/nginx-deployment.yaml
                    git commit -m "Update image to ${IMAGE_NAME}:${TAG} by Jenkins build #${TAG}" || echo "No changes to commit"
                    git push origin main
                    echo "✅ YAML changes pushed to repo"
                """
            }
        }

        stage('Trigger ArgoCD Sync') {
            steps {
                sh """
                    echo "=== Triggering ArgoCD Sync ==="

                    argocd login ${ARGOCD_SERVER} --username admin --password \$ARGOCD_PASSWORD --insecure

                    argocd app sync ${ARGOCD_APP} --grpc-web
                    echo "⏳ Waiting for ArgoCD to apply changes..."

                    argocd app wait ${ARGOCD_APP} --sync --health --grpc-web --timeout 180
                """
            }
        }

        stage('Verify Rollout (Kubernetes)') {
            steps {
                sh """
                    echo "=== Checking Deployment Rollout ==="
                    kubectl rollout status deployment/${ARGOCD_APP} --timeout=60s
                    kubectl get pods -l app=${ARGOCD_APP}
                """
            }
        }

    }

    post {
        success { echo "🎉 Deployment successfully synced via ArgoCD!" }
        failure { echo "❌ Deployment failed!" }
    }
}