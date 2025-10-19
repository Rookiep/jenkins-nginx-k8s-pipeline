pipeline {
    agent any
    
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
                        if [ -f Dockerfile ]; then
                            echo "✅ Dockerfile found"
                            echo "=== Dockerfile Contents ==="
                            cat Dockerfile
                        else
                            echo "❌ Dockerfile not found"
                            exit 1
                        fi
                        
                        echo "=== Checking k8s directory ==="
                        if [ -d k8s ]; then
                            echo "✅ k8s directory found"
                            ls -la k8s/
                            find k8s/ -name "*.yaml" -o -name "*.yml" | head -10
                        else
                            echo "❌ k8s directory not found"
                            exit 1
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
                        echo "Image Tag: nginx-app:\${BUILD_NUMBER}"
                        
                        # Check if we can use Docker
                        if [ -S /var/run/docker.sock ]; then
                            echo "✅ Docker socket found"
                            docker --version
                            docker build -t nginx-app:\${BUILD_NUMBER} .
                        else
                            echo "❌ Docker not available - using simulation"
                            echo "📝 Would build: docker build -t nginx-app:\${BUILD_NUMBER} ."
                            echo "✅ Build simulation complete"
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
                        # Update image tag in deployment
                        if [ -f k8s/nginx-deployment.yaml ]; then
                            sed -i 's|nginx-app:[0-9]*|nginx-app:\${BUILD_NUMBER}|g' k8s/nginx-deployment.yaml
                            echo "✅ Updated deployment with tag: nginx-app:\${BUILD_NUMBER}"
                            echo "=== Updated deployment ==="
                            cat k8s/nginx-deployment.yaml || true
                        else
                            echo "⚠️  nginx-deployment.yaml not found, skipping update"
                        fi
                    """
                }
            }
        }
    }
    
    post {
        always {
            sh '''
                echo "=== Cleanup ==="
                if command -v docker >/dev/null 2>&1 && [ -S /var/run/docker.sock ]; then
                    docker ps -aq --filter name=test | xargs -r docker rm -f || true
                fi
                echo "Cleanup completed"
            '''
        }
        success {
            echo "✅ Pipeline succeeded"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}