pipeline {
    agent any
    
    environment {
        IMAGE_TAG = "nginx-app:${env.BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Validate Project') {
            steps {
                script {
                    sh '''
                        echo "=== Project Validation ==="
                        echo "Build Number: ${BUILD_NUMBER}"
                        pwd
                        ls -la
                        
                        if [ -f "Dockerfile" ]; then
                            echo "✅ Dockerfile found"
                            echo "=== Dockerfile Contents ==="
                            cat Dockerfile
                        else
                            echo "❌ Dockerfile not found"
                            exit 1
                        fi
                    '''
                }
            }
        }
        
        stage('Build Info') {
            steps {
                script {
                    sh """
                        echo "=== Build Information ==="
                        echo "Image Tag: ${IMAGE_TAG}"
                        echo "Workspace: ${WORKSPACE}"
                        
                        # Create simple build info file
                        echo "build_image: ${IMAGE_TAG}" > build.txt
                        echo "build_time: \$(date)" >> build.txt
                        echo "status: ready" >> build.txt
                        cat build.txt
                    """
                }
            }
        }
        
        stage('Docker Test') {
            steps {
                script {
                    sh '''
                        echo "=== Docker Test ==="
                        # Check if docker command exists
                        if command -v docker > /dev/null 2>&1; then
                            echo "✅ Docker command found"
                            docker --version
                            
                            # Test Docker daemon connection
                            if docker ps > /dev/null 2>&1; then
                                echo "✅ Docker daemon is accessible"
                                echo "Building image..."
                                docker build -t nginx-app:test .
                                echo "Build completed"
                                docker rmi nginx-app:test
                            else
                                echo "⚠️ Cannot connect to Docker daemon"
                            fi
                        else
                            echo "⚠️ Docker command not found"
                        fi
                    '''
                }
            }
        }
    }
    
    post {
        always {
            echo "Pipeline finished: ${currentBuild.result}"
            sh 'rm -f build.txt 2>/dev/null || true'
        }
        success {
            echo "✅ Pipeline succeeded!"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}