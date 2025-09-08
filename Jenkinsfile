pipeline {
    agent any

    stages {
        stage('Code Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/mhanif45/KFC.git'
            }
        }

        stage('Docker Image Build') {
            steps {
                script {
                    docker.build("hanif040/kfc-static:${env.BUILD_NUMBER}")
                }
            }
        }

        stage('Docker Image Push') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credentials') {
                        docker.image("hanif040/kfc-static:${env.BUILD_NUMBER}").push()
                    }
                }
            }
        }
    }
}

