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
pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "nginx-app:${BUILD_NUMBER}"  // Use build number for versioning
        GIT_REPO = "https://github.com/Rookiep/jenkins-nginx-k8s-pipeline"  // Update with real Git URL
    }
    stages {
        stage('Build Docker Image') {
            steps { sh 'docker build -t ${DOCKER_IMAGE} .' }
        }
        stage('Push Docker Image') {
            steps {
                sh 'docker tag ${DOCKER_IMAGE} localhost:5000/${DOCKER_IMAGE}'
                sh 'docker push localhost:5000/${DOCKER_IMAGE}'
            }
        }
        stage('Update Git Repo') {
            steps {
                sh 'git clone ${GIT_REPO} temp-repo'  // Clone to temp
                sh '''
                    cd temp-repo
                    # Assume you update a deployment.yaml with new image tag here
                    sed -i "s|image: .*|image: localhost:5000/nginx-app:latest|g" k8s/nginx-deployment.yaml
                    git add .
                    git commit -m "Update image to ${BUILD_NUMBER}"
                    git push origin main
                '''
            }
        }
    }
    post { 
        success { echo 'Docker image pushed and Git updated. Argo CD will auto-sync.' } 
        always { sh 'rm -rf temp-repo' }  // Cleanup
    }
}