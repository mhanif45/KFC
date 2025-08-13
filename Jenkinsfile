pipeline {
    agent any

    environment {
        IMAGE_NAME = "hanif040/kfc-static"
        IMAGE_TAG  = "${BUILD_NUMBER}"
        IMAGE_FULL = "${IMAGE_NAME}:${IMAGE_TAG}"
    }

    stages {

        stage('Checkout') {
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
                echo "Pushing Docker image to Docker Hub..."
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', 
                                                 usernameVariable: 'DOCKER_USER', 
                                                 passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${IMAGE_FULL}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "Deploying to Kubernetes..."
                script {
                    def deployment = "kfc-static-deployment"
                    def containerName = sh(
                        script: "kubectl get deployment ${deployment} -o jsonpath='{.spec.template.spec.containers[0].name}'",
                        returnStdout: true
                    ).trim()

                    try {
                        sh """
                            kubectl set image deployment/${deployment} ${containerName}=${IMAGE_FULL} --record
                            kubectl rollout status deployment/${deployment} --timeout=180s
                        """
                        echo "Deployment updated successfully."
                    } catch (err) {
                        echo "Deployment failed, rolling back..."
                        sh "kubectl rollout undo deployment/${deployment}"
                        error "Deployment failed and rolled back."
                    }
                }
            }
        }

    }

    post {
        success {
            echo "Pipeline finished successfully."
        }
        failure {
            echo "Pipeline failed. Check logs above."
        }
    }
}

