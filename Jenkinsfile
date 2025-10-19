pipeline {
    agent any
    
    environment {
        IMAGE_TAG = "nginx-app:${env.BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh 'echo "✅ Code checked out"'
            }
        }
        
        stage('Verify Environment') {
            steps {
                script {
                    sh '''
                        echo "=== Environment Verification ==="
                        echo "Workspace: ${WORKSPACE}"
                        echo "User: $(whoami)"
                        pwd
                        ls -la
                        
                        # Check for Docker
                        echo "=== Docker Check ==="
                        if command -v docker > /dev/null 2>&1; then
                            echo "✅ Docker CLI found"
                            docker --version
                        else
                            echo "⚠️ Docker CLI not found in PATH"
                        # Check common locations
                            echo "Checking common Docker locations..."
                            ls -la /usr/bin/docker 2>/dev/null || echo "/usr/bin/docker not found"
                            ls -la /usr/local/bin/docker 2>/dev/null || echo "/usr/local/bin/docker not found"
                        fi
                        
                        # Check Docker socket
                        echo "=== Docker Socket Check ==="
                        if [ -S "/var/run/docker.sock" ]; then
                            echo "✅ Docker socket found"
                            ls -la /var/run/docker.sock
                        else
                            echo "❌ Docker socket not found"
                        fi
                        
                        # Check Dockerfile
                        echo "=== Dockerfile Check ==="
                        if [ -f "Dockerfile" ]; then
                            echo "✅ Dockerfile found"
                            echo "Contents:"
                            cat Dockerfile
                        else
                            echo "❌ Dockerfile not found"
                            exit 1
                        fi
                    '''
                }
            }
        }
        
        stage('Build Simulation') {
            steps {
                script {
                    sh """
                        echo "🚧 Build Simulation Mode"
                        echo "If Docker was working, we would run:"
                        echo "docker build -t ${IMAGE_TAG} ."
                        echo "docker images ${IMAGE_TAG}"
                        
                        # Create build simulation artifact
                        cat > build-info.yaml << EOF
build:
  image: ${IMAGE_TAG}
  timestamp: $(date)
  status: simulated
  docker_available: false
services:
  - name: nginx
    port: 80
    image: ${IMAGE_TAG}
EOF
                        echo "✅ Build simulation completed"
                        cat build-info.yaml
                    """
                }
            }
        }
    }
    
    post {
        always {
            echo "Pipeline result: ${currentBuild.result}"
            sh 'rm -f build-info.yaml 2>/dev/null || true'
        }
    }
}