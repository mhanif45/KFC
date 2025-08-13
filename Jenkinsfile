pipeline {
    agent any

    environment {
        DOCKER_USER = "hanif040"
        DOCKER_REPO = "kfc-static"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        IMAGE_FULL = "${DOCKER_USER}/${DOCKER_REPO}:${IMAGE_TAG}"
        GIT_REPO = "https://github.com/mhanif45/KFC.git"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: "${GIT_REPO}"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    set -e
                    echo "Building Docker image..."
                    docker build -t ${IMAGE_FULL} .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                        set -e
                        echo "Logging in to Docker Hub..."
                        echo "$PASS" | docker login -u "$USER" --password-stdin

                        echo "Pushing Docker image..."
                        docker push ${IMAGE_FULL}
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    set -e
                    echo "Updating Kubernetes deployment..."

                    # Get the first container name from deployment
                    CONTAINER=$(kubectl get deployment kfc-static-deployment \
                        -o jsonpath="{.spec.template.spec.containers[0].name}")

                    # Update image in deployment
                    kubectl set image deployment/kfc-static-deployment \
                        ${CONTAINER}=${IMAGE_FULL} --record

                    # Wait for rollout to finish
                    kubectl rollout status deployment/kfc-static-deployment --timeout=180s

                    echo "Deployment finished. Current image:"
                    kubectl get deployment kfc-static-deployment \
                        -o=jsonpath="{.spec.template.spec.containers[0].image}"; echo
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed. Check the logs above for errors."
        }
    }
}

