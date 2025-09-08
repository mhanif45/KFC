pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "hanif040/kfc-static"
    }

    stages {
        stage('Code Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/mhanif45/KFC.git'
            }
        }

        stage('Docker Image Build') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                }
            }
        }

        stage('Docker Image Push') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub') {
                        docker.image("${DOCKER_IMAGE}:${env.BUILD_NUMBER}").push()
                    }
                }
            }
        }

        stage('Kubernetes Deploy') {
            steps {
                script {
                    sh """
                    if kubectl --kubeconfig=/var/lib/jenkins/.kube/config get deployment kfc-static -n default; then
                        kubectl --kubeconfig=/var/lib/jenkins/.kube/config set image deployment/kfc-static \
                        kfc-website=${DOCKER_IMAGE}:${BUILD_NUMBER} -n default
                    else
                        kubectl --kubeconfig=/var/lib/jenkins/.kube/config create deployment kfc-static \
                        --image=${DOCKER_IMAGE}:${BUILD_NUMBER} -n default
                        kubectl --kubeconfig=/var/lib/jenkins/.kube/config expose deployment kfc-static \
                        --type=NodePort --port=80 -n default
                    fi
                    """
                }
            }
        }
    }
}

