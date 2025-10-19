stage('Build Docker Image') {
    steps {
        script {
            sh 'docker --version'
            sh "docker build -t $DOCKER_IMAGE ."
        }
    }
}

stage('Push Docker Image') {
    steps {
        script {
            withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
            }
            sh "docker tag $DOCKER_IMAGE $DOCKER_REGISTRY/$DOCKER_IMAGE"
            sh "docker push $DOCKER_REGISTRY/$DOCKER_IMAGE"
        }
    }
}

stage('Deploy to Kubernetes') {
    steps {
        script {
            sh "kubectl config use-context minikube"
            sh "kubectl get namespace $K8S_NAMESPACE || kubectl create namespace $K8S_NAMESPACE"
            sh "kubectl apply -n $K8S_NAMESPACE -f k8s/deployment.yaml"
            sh "kubectl apply -n $K8S_NAMESPACE -f k8s/service.yaml"
            sh "kubectl get all -n $K8S_NAMESPACE"
        }
    }
}
