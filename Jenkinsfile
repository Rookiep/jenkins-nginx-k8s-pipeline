pipeline {
    agent any

    environment {
        IMAGE_NAME = "nginx:latest"
        DOCKER_REGISTRY = "parthomazumdar"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/Rookiep/jenkins-nginx-k8s-pipeline.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker --version'
                    sh "docker build -t ${IMAGE_NAME} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    sh "docker login -u $DOCKER_REGISTRY"
                    sh "docker tag ${IMAGE_NAME} $DOCKER_REGISTRY/${IMAGE_NAME}"
                    sh "docker push $DOCKER_REGISTRY/${IMAGE_NAME}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh 'kubectl apply -f k8s/deployment.yaml'
                    sh 'kubectl apply -f k8s/service.yaml'
                }
            }
        }
    }

    post {
        failure {
            echo "Pipeline failed. Check logs!"
        }
        success {
            echo "Pipeline completed successfully!"
        }
    }
}