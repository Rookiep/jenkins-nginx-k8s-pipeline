pipeline {
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
        }
        stage('Push Docker Image') {
            steps {
                sh 'docker tag ${DOCKER_IMAGE} localhost:5000/${DOCKER_IMAGE}'
                sh 'docker push localhost:5000/${DOCKER_IMAGE}'  // Assumes local registry at :5000
            }
        }
        stage('Update Git Repo') {
            steps {
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
                '''
            }
        }
    }
    post {
        success { echo 'Pipeline succeeded! Argo CD should sync.' }
        always { sh 'docker rmi ${DOCKER_IMAGE} || true; rm -rf temp-repo' }
    }
}