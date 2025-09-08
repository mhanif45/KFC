pipeline {
    agent any

    environment {
        IMAGE_NAME = "hanif040/kfc-static"
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
                    docker.build("${IMAGE_NAME}:${env.BUILD_NUMBER}")
                }
            }
        }

        stage('Docker Image Push') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub') {
                        docker.image("${IMAGE_NAME}:${env.BUILD_NUMBER}").push()
                    }
                }
            }
        }

        stage('Kubernetes Deploy') {
            steps {
                script {
                    sh """
                        kubectl --kubeconfig=/var/lib/jenkins/.kube/config \
                        set image deployment/kfc-deployment \
                        kfc-website=${IMAGE_NAME}:${BUILD_NUMBER} -n default
                    """
                }
            }
        }
    }
}

