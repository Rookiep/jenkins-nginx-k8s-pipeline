pipeline {
    agent {
        docker {
            image 'docker:27.0' // Docker CLI image
            args '-v /var/run/docker.sock:/var/run/docker.sock' // Mount host Docker socket
        }
    }
    environment {
        DOCKER_IMAGE = "nginx-app:${env.BUILD_NUMBER}"
        GIT_REPO = "https://github.com/Rookiep/jenkins-nginx-k8s-pipeline.git"
    }
    stages {
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${env.DOCKER_IMAGE} ."
                sh "docker run -d -p 80:80 ${env.DOCKER_IMAGE}" // Optional test run
            }
        }
        stage('Push Docker Image') {
            steps {
                sh "docker tag ${env.DOCKER_IMAGE} localhost:5000/${env.DOCKER_IMAGE}"
                sh "docker push localhost:5000/${env.DOCKER_IMAGE}"
            }
        }
        stage('Update Git Repo') {
            steps {
                sh """
                    git clone ${env.GIT_REPO} temp-repo || true
                    cd temp-repo
                    sed -i "s|image: .*|image: localhost:5000/nginx-app:${env.BUILD_NUMBER}|g" k8s/nginx-deployment.yaml
                    git config --global user.email "jenkins@example.com"
                    git config --global user.name "Jenkins"
                    git add .
                    git commit -m "Update image to ${env.BUILD_NUMBER}" || true
                    git push origin main || true
                """
            }
        }
    }
    post {
        success {
            echo 'Pipeline succeeded! Docker image pushed and Git updated. Argo CD will auto-sync.'
        }
        always {
            sh "docker rmi ${env.DOCKER_IMAGE} || true"
            sh "rm -rf temp-repo"
        }
    }
}
