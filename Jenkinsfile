pipeline {
    agent any

    environment {
        IMAGE_NAME = "hanif040/kfc-static"
        IMAGE_TAG = "latest"
        IMAGE_FULL = "${IMAGE_NAME}:${IMAGE_TAG}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo "Checking out source code..."
                git branch: 'main', url: 'https://github.com/mhanif45/KFC.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image ${IMAGE_FULL}..."
                sh """
                    docker build -t ${IMAGE_FULL} .
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                echo "Logging in and pushing Docker image..."
                withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE_FULL}
                        docker logout
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "Deploying to Kubernetes..."
                sh """
                    CONTAINER=\$(kubectl get deployment kfc-static-deployment -o jsonpath='{.spec.template.spec.containers[0].name}')
                    kubectl set image deployment/kfc-static-deployment \${CONTAINER}=${IMAGE_FULL} --record
                    kubectl rollout status deployment/kfc-static-deployment --timeout=180s
                    echo "Deployment complete. Current image: \${IMAGE_FULL}"
                """
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully."
        }
        failure {
            echo "Pipeline failed. Check logs for details."
        }
    }
}

