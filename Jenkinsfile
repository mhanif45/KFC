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

        stage('Kubernetes Deploy') {
    steps {
        withCredentials([string(credentialsId: 'Kube_config', variable: 'KUBECONFIG_BASE64')]) {
            sh '''
                echo "$KUBECONFIG_BASE64" | base64 --decode > kubeconfig
                kubectl --kubeconfig=kubeconfig set image deployment/kfc-deployment kfc-website=hanif040/kfc-static:${BUILD_NUMBER} -n default
                rm -f kubeconfig
            '''
        }
    }
}
    }
}

