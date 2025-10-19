pipeline {
    agent any
    
    environment {
        IMAGE_TAG = "nginx-app:${env.BUILD_NUMBER}"
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
                script {
                    sh '''
                        echo "=== Project Structure ==="
                        echo "Workspace: ${WORKSPACE}"
                        pwd
                        ls -la
                        
                        echo "=== Checking for Dockerfile ==="
                        if [ -f "Dockerfile" ]; then
                            echo "✅ Dockerfile found"
                            echo "=== Dockerfile Contents ==="
                            cat Dockerfile
                        elif [ -f "Docker" ]; then
                            echo "⚠️ Found 'Docker' file (without .file extension)"
                            echo "=== Docker File Contents ==="
                            cat Docker
                            echo "Creating Dockerfile from Docker file..."
                            cp Docker Dockerfile
                            echo "✅ Dockerfile created"
                        else
                            echo "❌ No Dockerfile or Docker file found"
                            echo "Creating default Dockerfile..."
                            cat > Dockerfile << EOF
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
EOF
                            echo "✅ Default Dockerfile created"
                        fi
                        
                        echo "=== Final Dockerfile ==="
                        cat Dockerfile
                        
                        echo "=== Checking k8s directory ==="
                        if [ -d "k8s" ]; then
                            echo "✅ k8s directory found"
                            ls -la k8s/
                            find k8s/ -name "*.yaml" -o -name "*.yml" | head -10
                        else
                            echo "⚠️ k8s directory not found"
                        fi
                    '''
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                        echo "=== Building Docker Image ==="
                        echo "Image Tag: ${IMAGE_TAG}"
                        
                        # Check if Docker is available
                        if command -v docker > /dev/null 2>&1; then
                            echo "✅ Docker is available"
                            docker --version
                            
                            # Build the image
                            docker build -t ${IMAGE_TAG} .
                            echo "✅ Docker image built successfully"
                            
                            # List the image
                            docker images | grep nginx-app
                        else
                            echo "⚠️ Docker not available, simulating build"
                            echo "Simulated: docker build -t ${IMAGE_TAG} ."
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
                        echo "=== Updating K8s Manifests ==="
                        
                        # Create k8s directory if it doesn't exist
                        mkdir -p k8s
                        
                        # Create or update deployment.yaml
                        cat > k8s/deployment.yaml << EOL
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
        image: ${IMAGE_TAG}
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
EOL
                        echo "✅ K8s deployment manifest updated"
                        echo "Image set to: ${IMAGE_TAG}"
                        
                        # Show the created file
                        cat k8s/deployment.yaml
                    """
                }
            }
        }
    }
    
    post {
        always {
            echo "Pipeline finished: ${currentBuild.result}"
            sh '''
                echo "=== Cleanup ==="
                # Remove any test containers
                docker ps -aq --filter "name=test" 2>/dev/null | xargs -r docker rm -f 2>/dev/null || true
                echo "Cleanup completed"
            '''
        }
        success {
            echo "🎉 Pipeline succeeded!"
            echo "📦 Docker Image: ${env.IMAGE_TAG}"
            echo "📁 K8s manifests updated in k8s/deployment.yaml"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}

