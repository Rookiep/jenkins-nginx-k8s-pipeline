pipeline {
<<<<<<< HEAD
    agent {
        docker {
            image 'docker:27.0-dind'  // DinD image with built-in daemon
            args '--privileged -v /var/run/docker.sock:/var/run/docker.sock'  // Privileged for daemon; socket optional
        }
    }
    environment {
        DOCKER_IMAGE = "nginx-app:${BUILD_NUMBER}"
        GIT_REPO = "https://github.com/yourusername/jenkins-nginx-k8s-pipeline.git"  // Update with YOUR GitHub repo URL
    }
    stages {
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE} .'
                sh 'docker run -d -p 80:80 ${DOCKER_IMAGE}'  // Optional: Quick test run
            }
=======
    agent any
    environment {
        DOCKER_IMAGE = "nginx-app:${BUILD_NUMBER}"  // Use build number for versioning
        GIT_REPO = "https://github.com/Rookiep/jenkins-nginx-k8s-pipeline"  // Update with real Git URL
    }
    stages {
        stage('Build Docker Image') {
            steps { sh 'docker build -t ${DOCKER_IMAGE} .' }
>>>>>>> 742366a66ff1236afc3926551206ab0b1c3a063e
        }
        stage('Push Docker Image') {
            steps {
                sh 'docker tag ${DOCKER_IMAGE} localhost:5000/${DOCKER_IMAGE}'
<<<<<<< HEAD
                sh 'docker push localhost:5000/${DOCKER_IMAGE}'  // Assumes local registry at :5000
=======
                sh 'docker push localhost:5000/${DOCKER_IMAGE}'
>>>>>>> 742366a66ff1236afc3926551206ab0b1c3a063e
            }
        }
        stage('Update Git Repo') {
            steps {
<<<<<<< HEAD
                sh '''
                    git clone ${GIT_REPO} temp-repo || true
                    cd temp-repo
                    # Update image tag in manifests (example for deployment.yaml)
                    sed -i "s|image: .*|image: localhost:5000/nginx-app:${BUILD_NUMBER}|g" k8s/nginx-deployment.yaml
                    git config --global user.email "jenkins@example.com"
                    git config --global user.name "Jenkins"
                    git add .
                    git commit -m "Update image to ${BUILD_NUMBER}" || true
                    git push || true  # Add credentials if private repo
=======
                sh 'git clone ${GIT_REPO} temp-repo'  // Clone to temp
                sh '''
                    cd temp-repo
                    # Assume you update a deployment.yaml with new image tag here
                    sed -i "s|image: .*|image: localhost:5000/nginx-app:latest|g" k8s/nginx-deployment.yaml
                    git add .
                    git commit -m "Update image to ${BUILD_NUMBER}"
                    git push origin main
>>>>>>> 742366a66ff1236afc3926551206ab0b1c3a063e
                '''
            }
        }
    }
<<<<<<< HEAD
    post {
        success { echo 'Pipeline succeeded! Argo CD should sync.' }
        always { sh 'docker rmi ${DOCKER_IMAGE} || true; rm -rf temp-repo' }
=======
    post { 
        success { echo 'Docker image pushed and Git updated. Argo CD will auto-sync.' } 
        always { sh 'rm -rf temp-repo' }  // Cleanup
>>>>>>> 742366a66ff1236afc3926551206ab0b1c3a063e
    }
}