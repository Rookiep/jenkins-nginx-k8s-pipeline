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
        
        stage('Setup Docker') {
            steps {
                script {
                    sh '''
                        echo "Setting up Docker client..."
                        apt-get update
                        apt-get install -y docker.io
                        docker --version
                    '''
                }
            }
        }
        
        stage('Build Image') {
            steps {
                script {
                    sh """
                        echo "Building Docker image: ${IMAGE_TAG}"
                        docker build -t ${IMAGE_TAG} .
                        docker images | grep nginx-app
                    """
                }
            }
        }
        
        stage('Test Image') {
            steps {
                script {
                    sh """
                        echo "Testing the image..."
                        docker run --rm ${IMAGE_TAG} nginx -v
                        echo "✅ Image test passed"
                    """
                }
            }
        }
    }
    
    post {
        always {
            echo "Pipeline finished: ${currentBuild.result}"
        }
        success {
            echo "✅ SUCCESS: Docker pipeline is working!"
        }
    }
}