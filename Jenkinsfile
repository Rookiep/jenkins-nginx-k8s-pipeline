pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "nginx:latest"
        DOCKER_REGISTRY = "parthomazumdar"  // replace with your DockerHub username
        K8S_NAMESPACE = "nginx-demo"
        K8S_DEPLOYMENT = "nginx-deployment"
        K8S_SERVICE = "nginx-service"
    }

    stages {

        stage('Checkout SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/Rookiep/jenkins-nginx-k8s-pipeline'
            }
        }

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
                    // DockerHub login using credentials stored in Jenkins
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    }

                    // Tag and push
                    sh "docker tag $DOCKER_IMAGE $DOCKER_REGISTRY/$DOCKER_IMAGE"
                    sh "docker push $DOCKER_REGISTRY/$DOCKER_IMAGE"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Use Minikube context (or your k8s context)
                    sh "kubectl config use-context minikube"

                    // Create namespace if it doesn't exist
                    sh "kubectl get namespace $K8S_NAMESPACE || kubectl create namespace $K8S_NAMESPACE"

                    // Apply Deployment manifest
                    sh """
                    kubectl apply -n $K8S_NAMESPACE -f - <<EOF
                    apiVersion: apps/v1
                    kind: Deployment
                    metadata:
                      name: $K8S_DEPLOYMENT
                    spec:
                      replicas: 1
                      selector:
                        matchLabels:
                          app: nginx
                      template:
                        metadata:
                          labels:
                            app: nginx
                        spec:
                          containers:
                          - name: nginx
                            image: $DOCKER_REGISTRY/$DOCKER_IMAGE
                            ports:
                            - containerPort: 80
                    EOF
                    """

                    // Apply Service manifest
                    sh """
                    kubectl apply -n $K8S_NAMESPACE -f - <<EOF
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: $K8S_SERVICE
                    spec:
                      selector:
                        app: nginx
                      ports:
                      - protocol: TCP
                        port: 80
                        targetPort: 80
                      type: NodePort
                    EOF
                    """

                    // Show deployment and service status
                    sh "kubectl get all -n $K8S_NAMESPACE"
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline finished successfully! Nginx deployed to Kubernetes."
        }
        failure {
            echo "Pipeline failed. Check logs."
        }
    }
}
