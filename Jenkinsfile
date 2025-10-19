pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build and Test') {
            steps {
                script {
                    sh '''
                        echo "Building Docker image..."
                        docker build -t nginx-app:${BUILD_NUMBER} .
                        echo "Image built successfully"
                        
                        # Simple test - just check if Dockerfile exists and image builds
                        if [ -f "Dockerfile" ]; then
                            echo "Dockerfile found - build successful"
                        else
                            echo "No Dockerfile found"
                            exit 1
                        fi
                    '''
                }
            }
        }
    }
    
    post {
        always {
            echo "Pipeline finished: ${currentBuild.result}"
        }
    }
}