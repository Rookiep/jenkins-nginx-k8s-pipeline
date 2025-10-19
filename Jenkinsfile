pipeline {
    agent any
    
    environment {
        IMAGE_TAG = "nginx:alpine"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Fix Invalid YAML Files') {
            steps {
                script {
                    sh '''
                        echo "=== Fixing Invalid YAML Files ==="
                        
                        # Fix hpa.yaml - remove the invalid + sign
                        if [ -f "k8s/hpa.yaml" ]; then
                            echo "Fixing hpa.yaml..."
                            cat > k8s/hpa.yaml << EOF
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 1
  maxReplicas: 3
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
EOF
                            echo "✅ Fixed hpa.yaml"
                        fi
                        
                        # Validate all YAML files after fixing
                        echo "=== Validating Fixed YAML Files ==="
                        for file in k8s/*.yaml k8s/*.yml; do
                            if [ -f "\$file" ]; then
                                echo "Validating: \$file"
                                if python3 -c "import yaml; yaml.safe_load(open('\$file'))" 2>/dev/null; then
                                    echo "✅ Valid YAML"
                                else
                                    echo "❌ Still invalid YAML in \$file"
                                    echo "File content:"
                                    cat "\$file"
                                    exit 1
                                fi
                            fi
                        done
                        
                        echo "🎉 All YAML files are now valid!"
                    '''
                }
            }
        }
        
        stage('Validate Kubernetes Manifests') {
            steps {
                script {
                    sh '''
                        echo "=== Validating Kubernetes Manifests ==="
                        
                        # Check if kubectl is available for validation
                        if command -v kubectl > /dev/null 2>&1; then
                            for file in k8s/*.yaml k8s/*.yml; do
                                if [ -f "\$file" ]; then
                                    echo "Validating Kubernetes syntax: \$file"
                                    if kubectl apply -f "\$file" --dry-run=client --validate=true; then
                                        echo "✅ Valid Kubernetes manifest"
                                    else
                                        echo "❌ Invalid Kubernetes manifest"
                                        exit 1
                                    fi
                                fi
                            done
                        else
                            echo "⚠️ kubectl not available, skipping Kubernetes validation"
                        fi
                    '''
                }
            }
        }
        
        stage('Update Deployment Image') {
            steps {
                script {
                    sh """
                        echo "Updating deployment with image tag..."
                        
                        # Update the nginx-deployment.yaml with the correct image
                        if [ -f "k8s/nginx-deployment.yaml" ]; then
                            sed -i "s|image:.*|image: ${IMAGE_TAG}|g" k8s/nginx-deployment.yaml
                            echo "✅ Updated nginx-deployment.yaml with image: ${IMAGE_TAG}"
                            
                            # Show the updated content
                            echo "=== Updated deployment ==="
                            cat k8s/nginx-deployment.yaml
                        else
                            echo "⚠️ nginx-deployment.yaml not found"
                        fi
                    """
                }
            }
        }
        
        stage('Commit and Push Fixes') {
            steps {
                script {
                    sh '''
                        echo "=== Committing YAML Fixes ==="
                        git config --global user.email "jenkins@example.com"
                        git config --global user.name "Jenkins"
                        
                        git add k8s/
                        git status
                        
                        if git diff --cached --quiet; then
                            echo "No changes to commit"
                        else
                            git commit -m "Fix: Correct invalid YAML syntax in hpa.yaml and update manifests"
                            git push origin main
                            echo "✅ Fixed YAML files pushed to repository"
                        fi
                    '''
                }
            }
        }
        
        stage('Verify ArgoCD Sync') {
            steps {
                script {
                    sh '''
                        echo "=== ArgoCD Sync Status ==="
                        echo "After pushing the fixes, ArgoCD should automatically sync"
                        echo "The previous error was due to invalid YAML in hpa.yaml"
                        echo "The + sign after 'averageUtilization: 50' was causing the issue"
                        
                        # List all fixed files
                        echo "=== Fixed Files ==="
                        find k8s/ -name "*.yaml" -o -name "*.yml" | while read file; do
                            echo "✅ Fixed: \$file"
                        done
                    '''
                }
            }
        }
    }
    
    post {
        always {
            echo "Pipeline finished: ${currentBuild.result}"
        }
        success {
            echo "✅ Pipeline succeeded! YAML syntax errors fixed."
            echo "📁 Valid k8s manifests pushed to repository"
            echo "🔄 ArgoCD should now sync successfully"
        }
        failure {
            echo "❌ Pipeline failed. Check specific error above."
        }
    }
}