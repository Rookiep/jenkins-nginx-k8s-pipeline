stage('Build Docker Image') {
    steps {
        script {
            sh """
                echo "=== Simulating Docker Build ==="
                echo "Image Tag: ${DOCKER_TAG}"
                echo "📝 Docker build command: docker build -t ${DOCKER_TAG} ."
                echo "✅ Build simulation complete - continuing pipeline"
            """
        }
    }
}