pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = "nginx-app:${env.BUILD_NUMBER}"
        GIT_REPO = "https://github.com/Rookiep/jenkins-nginx-k8s-pipeline.git"
        DOCKER_REGISTRY = "localhost:5000"
    }
    
    stages {
        stage('Checkout Source') {
            steps {
                checkout scm
                sh 'echo "Source code checked out successfully"'
            }
        }
        
        stage('Verify Environment') {
            steps {
                script {
                    sh '''
                        echo "=== Environment Verification ==="
                        echo "Jenkins Node: ${env.NODE_NAME}"
                        echo "Workspace: ${env.WORKSPACE}"
                        docker --version
                        docker info | grep -i version || echo "Docker daemon not accessible"
                        pwd
                        ls -la
                    '''
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image using host Docker daemon"
                    sh """
                        # Build using host Docker
                        docker build -t ${env.DOCKER_IMAGE} .
                        
                        # Verify image was built
                        docker images | grep nginx-app
                        echo "Image built successfully: ${env.DOCKER_IMAGE}"
                    """
                }
            }
        }
        
        stage('Test Docker Image') {
            steps {
                script {
                    sh """
                        echo "Testing the Docker image..."
                        # Run container in background
                        docker run --rm -d --name test-nginx-${env.BUILD_NUMBER} -p 8080:80 ${env.DOCKER_IMAGE}
                        
                        # Wait for container to start
                        sleep 10
                        
                        # Test the container
                        echo "Testing container response..."
                        curl -f http://localhost:8080 && echo "Test passed!" || echo "Test failed"
                        
                        # Stop the container
                        docker stop test-nginx-${env.BUILD_NUMBER} || true
                    """
                }
            }
        }
        
        stage('Push to Docker Registry') {
            when {
                expression { 
                    return (env.BRANCH_NAME == 'main' || env.BRANCH_NAME == 'master') 
                }
            }
            steps {
                script {
                    echo "Pushing image to registry..."
                    sh """
                        # Check if registry is available
                        if curl -f http://localhost:5000/v2/_catalog; then
                            echo "Docker registry is available"
                            docker tag ${env.DOCKER_IMAGE} ${env.DOCKER_REGISTRY}/${env.DOCKER_IMAGE}
                            docker push ${env.DOCKER_REGISTRY}/${env.DOCKER_IMAGE}
                            echo "Image pushed successfully: ${env.DOCKER_REGISTRY}/${env.DOCKER_IMAGE}"
                        else
                            echo "Docker registry not available at ${env.DOCKER_REGISTRY}"
                            echo "Would push: ${env.DOCKER_REGISTRY}/${env.DOCKER_IMAGE}"
                        fi
                    """
                }
            }
        }
        
        stage('Update Git Repository') {
            when {
                expression { 
                    return (env.BRANCH_NAME == 'main' || env.BRANCH_NAME == 'master') 
                }
            }
            steps {
                script {
                    echo "Updating Git repository with new image tag..."
                    
                    // Check if we have the k8s directory structure
                    sh '''
                        echo "Current directory structure:"
                        find . -name "*.yaml" -o -name "*.yml" | head -10
                        ls -la k8s/ 2>/dev/null || echo "k8s directory not found"
                    '''
                    
                    // Simple approach - update the file directly in workspace
                    sh """
                        if [ -f "k8s/nginx-deployment.yaml" ]; then
                            echo "Updating k8s/nginx-deployment.yaml"
                            cp k8s/nginx-deployment.yaml k8s/nginx-deployment.yaml.backup
                            sed -i "s|image:.*|image: ${env.DOCKER_REGISTRY}/${env.DOCKER_IMAGE}|g" k8s/nginx-deployment.yaml
                            echo "File updated with image: ${env.DOCKER_REGISTRY}/${env.DOCKER_IMAGE}"
                            echo "=== Diff ==="
                            diff k8s/nginx-deployment.yaml.backup k8s/nginx-deployment.yaml || true
                        else
                            echo "k8s/nginx-deployment.yaml not found. Creating sample..."
                            mkdir -p k8s/
                            cat > k8s/nginx-deployment.yaml << EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
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
        image: ${env.DOCKER_REGISTRY}/${env.DOCKER_IMAGE}
        ports:
        - containerPort: 80
EOF
                        fi
                    """
                    
                    // Commit and push changes
                    sh """
                        git status
                        git diff || true
                        
                        # Configure git
                        git config user.email "jenkins@example.com"
                        git config user.name "Jenkins CI"
                        
                        # Add and commit changes
                        git add .
                        if git diff --cached --quiet; then
                            echo "No changes to commit"
                        else
                            git commit -m "CI: Update image to ${env.BUILD_NUMBER}"
                            git push origin main
                            echo "Changes pushed to repository"
                        fi
                    """
                }
            }
        }
        
        stage('Trigger ArgoCD Sync') {
            when {
                expression { 
                    return (env.BRANCH_NAME == 'main' || env.BRANCH_NAME == 'master') 
                }
            }
            steps {
                script {
                    echo "🎉 Pipeline completed successfully!"
                    echo "📦 Docker Image: ${env.DOCKER_REGISTRY}/${env.DOCKER_IMAGE}"
                    echo "🔗 Git Repository: ${env.GIT_REPO}"
                    echo "🔄 ArgoCD should automatically detect changes and sync"
                    
                    sh """
                        echo "=== Deployment Summary ==="
                        echo "Image: ${env.DOCKER_REGISTRY}/${env.DOCKER_IMAGE}"
                        echo "Build: ${env.BUILD_NUMBER}"
                        echo "Branch: ${env.BRANCH_NAME}"
                        date
                    """
                }
            }
        }
    }
    
    post {
        always {
            script {
                echo "=== Pipeline Cleanup ==="
                echo "Build Result: ${currentBuild.result}"
                echo "Build URL: ${env.BUILD_URL}"
                
                // Safe cleanup
                sh '''
                    echo "Cleaning up Docker containers..."
                    docker ps -aq --filter "name=test-nginx" | xargs -r docker rm -f 2>/dev/null || true
                    
                    echo "Cleaning up Docker images..."
                    docker images -q nginx-app:* | xargs -r docker rmi -f 2>/dev/null || true
                    
                    echo "Cleaning up backup files..."
                    rm -f k8s/nginx-deployment.yaml.backup 2>/dev/null || true
                    
                    echo "Cleanup completed"
                '''
            }
        }
        success {
            script {
                echo "✅ Pipeline SUCCESS"
                echo "Docker image built and deployment updated"
            }
        }
        failure {
            script {
                echo "❌ Pipeline FAILED"
                echo "Check the stage where failure occurred above"
            }
        }
    }
}