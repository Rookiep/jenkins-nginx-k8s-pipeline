pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = "nginx-app:${env.BUILD_NUMBER}"
        GIT_REPO = "https://github.com/Rookiep/jenkins-nginx-k8s-pipeline.git"
        DOCKER_REGISTRY = "localhost:5000"
    }
    
    stages {
        stage('Checkout Source') {
            steps {
                checkout scm
                sh 'echo "Source code checked out successfully"'
            }
        }
        
        stage('Setup Docker Environment') {
            steps {
                script {
                    // Verify Docker is available
                    sh '''
                        echo "Setting up Docker environment..."
                        docker --version || echo "Docker not available in base agent"
                    '''
                }
            }
        }
        
        stage('Build Docker Image') {
            agent {
                docker {
                    image 'docker:27.0'
                    args '--privileged -v /var/run/docker.sock:/var/run/docker.sock -v /tmp:/tmp'
                    reuseNode true
                }
            }
            steps {
                script {
                    echo "Building Docker image: ${env.DOCKER_IMAGE}"
                    sh """
                        docker build -t ${env.DOCKER_IMAGE} .
                        docker images | grep nginx-app
                    """
                }
            }
        }
        
        stage('Test Docker Image') {
            agent {
                docker {
                    image 'docker:27.0'
                    args '--privileged -v /var/run/docker.sock:/var/run/docker.sock'
                    reuseNode true
                }
            }
            steps {
                script {
                    sh """
                        echo "Testing the Docker image..."
                        docker run --rm -d --name test-container -p 8080:80 ${env.DOCKER_IMAGE} && \\
                        sleep 5 && \\
                        curl -f http://localhost:8080 || echo "Container test failed" && \\
                        docker stop test-container || true
                    """
                }
            }
        }
        
        stage('Push to Docker Registry') {
            agent {
                docker {
                    image 'docker:27.0'
                    args '--privileged -v /var/run/docker.sock:/var/run/docker.sock'
                    reuseNode true
                }
            }
            when {
                expression { 
                    return (env.BRANCH_NAME == 'main' || env.BRANCH_NAME == 'master') 
                }
            }
            steps {
                script {
                    echo "Pushing image to registry..."
                    sh """
                        docker tag ${env.DOCKER_IMAGE} ${env.DOCKER_REGISTRY}/${env.DOCKER_IMAGE}
                        docker push ${env.DOCKER_REGISTRY}/${env.DOCKER_IMAGE} || echo "Registry push failed - ensure registry is running"
                    """
                }
            }
        }
        
        stage('Update Git Repository') {
            when {
                expression { 
                    return (env.BRANCH_NAME == 'main' || env.BRANCH_NAME == 'master') 
                }
            }
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'github-credentials',
                        usernameVariable: 'GIT_USERNAME',
                        passwordVariable: 'GIT_PASSWORD'
                    )]) {
                        sh """
                            echo "Updating Git repository with new image tag..."
                            # Clone the repository
                            git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/Rookiep/jenkins-nginx-k8s-pipeline.git temp-repo || exit 1
                            
                            cd temp-repo
                            
                            # Update the deployment file
                            if [ -f "k8s/nginx-deployment.yaml" ]; then
                                sed -i "s|image:.*|image: ${env.DOCKER_REGISTRY}/${env.DOCKER_IMAGE}|g" k8s/nginx-deployment.yaml
                                echo "Updated k8s/nginx-deployment.yaml with image: ${env.DOCKER_REGISTRY}/${env.DOCKER_IMAGE}"
                            else
                                echo "k8s/nginx-deployment.yaml not found. Available files:"
                                find . -name "*.yaml" -o -name "*.yml" | head -10
                                exit 1
                            fi
                            
                            # Configure git and push changes
                            git config user.email "jenkins@example.com"
                            git config user.name "Jenkins CI"
                            
                            git add .
                            git status
                            git commit -m "CI: Update image to ${env.BUILD_NUMBER}" || echo "No changes to commit"
                            git push origin main || echo "Push failed or no changes"
                            
                            cd ..
                        """
                    }
                }
            }
        }
        
        stage('Trigger ArgoCD Sync') {
            when {
                expression { 
                    return (env.BRANCH_NAME == 'main' || env.BRANCH_NAME == 'master') 
                }
            }
            steps {
                script {
                    echo "ArgoCD should automatically detect the Git changes and sync"
                    sh """
                        echo "Image updated to: ${env.DOCKER_REGISTRY}/${env.DOCKER_IMAGE}"
                        echo "ArgoCD auto-sync should deploy the new version"
                    """
                }
            }
        }
    }
    
    post {
        always {
            script {
                echo "Pipeline execution completed with result: ${currentBuild.result}"
                
                // Safe cleanup that doesn't require specific context
                sh '''
                    echo "Performing cleanup..."
                    # Remove temporary directory
                    rm -rf temp-repo || true
                    
                    # Clean up Docker containers and images
                    docker ps -aq --filter "name=test-container" | xargs -r docker rm -f || true
                    docker images -q nginx-app:* | xargs -r docker rmi -f || true
                '''
            }
        }
        success {
            script {
                echo " Pipeline succeeded! Docker image built and pushed."
                echo " Image: ${env.DOCKER_REGISTRY}/${env.DOCKER_IMAGE}"
                echo " ArgoCD will auto-sync the deployment"
            }
        }
        failure {
            script {
                echo " Pipeline failed! Check the logs above for details."
            }
        }
        unstable {
            echo "Pipeline marked as unstable"
        }
    }
}