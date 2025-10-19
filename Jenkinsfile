pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh 'echo "Code checked out successfully"'
            }
        }
        stage('Test') {
            steps {
                sh '''
                    echo "Testing environment..."
                    pwd
                    ls -la
                    if [ -f "Dockerfile" ]; then
                        echo "Dockerfile found - project is valid"
                        cat Dockerfile
                    else
                        echo "ERROR: Dockerfile not found"
                        exit 1
                    fi
                '''
            }
        }
        stage('Build Info') {
            steps {
                sh """
                    echo "Build Number: ${env.BUILD_NUMBER}"
                    echo "Would build: nginx-app:${env.BUILD_NUMBER}"
                """
            }
        }
    }
    post {
        always {
            echo "Pipeline finished: ${currentBuild.result}"
        }
    }
}