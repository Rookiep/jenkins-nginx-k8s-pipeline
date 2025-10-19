pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "nginx:latest"
        GIT_REPO = "http://localhost/git/nginx-k8s.git"
    }
    stages {
        stage('Build Docker Image') { steps { sh 'docker build -t $DOCKER_IMAGE .' } }
        stage('Push Docker Image') { steps { sh 'docker tag $DOCKER_IMAGE localhost:5000/$DOCKER_IMAGE'; sh 'docker push localhost:5000/$DOCKER_IMAGE' } }
        stage('Update Git Repo') { steps { sh 'git clone $GIT_REPO'; sh 'cd nginx-k8s && git add . && git commit -m "Update image" && git push' } }
    }
    post { success { echo 'Docker image pushed and Git updated. Argo CD will auto-sync.' } }
}