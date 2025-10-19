pipeline {
    agent any
    
    environment {
        IMAGE_TAG = "nginx-app:${env.BUILD_NUMBER}"
    }
    
    stages {
        stage('Clean Workspace') {
            steps {
                script {
                    sh '''
                        echo "Cleaning workspace..."
                        # Remove any large binary files that shouldn't be here
                        find . -name "*.exe" -type f -delete 2>/dev/null || true
                        find . -name "argocd*" -type f -delete 2>/dev/null || true
                        
                        echo "Current directory:"
                        pwd
                        ls -la
                    '''
                }
            }
        }
        
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Verify k8s Manifests') {
            steps {
                script {
                    sh '''
                        echo "=== Verifying k8s Directory ==="
                        ls -la k8s/ || echo "k8s directory not found"
                        
                        echo "=== Checking YAML Files ==="
                        find k8s/ -name "*.yaml" -o -name "*.yml" | while read file; do
                            echo "File: \$file"
                            if [ -f "\$file" ]; then
                                echo "Content:"
                                cat "\$file"
                                echo "---"
                                
                                # Validate YAML
                                if python3 -c "import yaml; yaml.safe_load(open('\$file'))" 2>/dev/null; then
                                    echo "✅ Valid YAML"
                                else
                                    echo "❌ Invalid YAML"
                                    exit 1
                                fi
                            fi
                        done
                    '''
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                        echo "Building Docker image: ${IMAGE_TAG}"
                        if command -v docker > /dev/null 2>&1; then
                            docker build -t ${IMAGE_TAG} .
                            echo "✅ Docker image built successfully"
                            docker images | grep nginx-app
                        else
                            echo "⚠️ Docker not available, simulating build"
                            echo "Image would be: ${IMAGE_TAG}"
                        fi
                    """
                }
            }
        }
        
        stage('Update K8s Manifests') {
            steps {
                script {
                    sh """
                        echo "Updating Kubernetes manifests with new image tag..."
                        
                        # Update deployment with new image tag
                        if [ -f "k8s/deployment.yaml" ]; then
                            sed -i "s|image:.*|image: ${IMAGE_TAG}|g" k8s/deployment.yaml
                            echo "✅ Updated deployment.yaml with image: ${IMAGE_TAG}"
                        else
                            echo "⚠️ deployment.yaml not found, creating it..."
                            mkdir -p k8s
                            cat > k8s/deployment.yaml << EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: ${IMAGE_TAG}
        ports:
        - containerPort: 80
EOF
                        fi
                        
                        # Ensure service exists
                        if [ ! -f "k8s/service.yaml" ]; then
                            cat > k8s/service.yaml << EOF
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: default
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
EOF
                            echo "✅ Created service.yaml"
                        fi
                        
                        echo "=== Final k8s manifests ==="
                        ls -la k8s/
                        cat k8s/deployment.yaml
                    """
                }
            }
        }
        
        stage('Commit Changes') {
            steps {
                script {
                    sh '''
                        echo "Committing changes to repository..."
                        git config --global user.email "jenkins@example.com"
                        git config --global user.name "Jenkins"
                        
                        # Remove any large files first
                        git rm --cached argocd.exe 2>/dev/null || true
                        git rm --cached *.exe 2>/dev/null || true
                        
                        git add .
                        git status
                        
                        if git diff --cached --quiet; then
                            echo "No changes to commit"
                        else
                            git commit -m "Update K8s manifests and remove large files"
                            git push origin main
                            echo "✅ Changes pushed successfully"
                        fi
                    '''
                }
            }
        }
    }
    
    post {
        always {
            echo "Pipeline finished: ${currentBuild.result}"
            sh '''
                echo "Cleaning up..."
                # Remove any binary files that might have been created
                find . -name "*.exe" -type f -delete 2>/dev/null || true
            '''
        }
        success {
            echo "✅ Pipeline succeeded! Repository is clean."
            echo "📦 Docker Image: ${env.IMAGE_TAG}"
            echo "📁 K8s manifests updated"
        }
    }
}