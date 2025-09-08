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
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub') {
                        docker.image("hanif040/kfc-static:${env.BUILD_NUMBER}").push()
                    }
                }
            }
        }

stage('Deploy to Kubernetes') {
    steps {
        script {
            withCredentials([credentialsId: 'k8s-cluster']) {
                sh """
                kubectl set image deployment/kfc-deployment kfc-website=hanif040/kfc-static:${env.BUILD_NUMBER} -n default
                kubectl rollout status deployment/kfc-deployment -n default
                """
            }
        }
    }
}
    }
}

