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
            // Use the kube_config credential
            withCredentials([string(credentialsId: 'kube_config', variable: 'KUBECONFIG_CONTENT')]) {
                // Save the kubeconfig to a file
                writeFile file: 'kubeconfig', text: env.KUBECONFIG_CONTENT

                // Use kubectl with the kubeconfig file
                sh 'kubectl --kubeconfig=kubeconfig set image deployment/kfc-deployment kfc-website=hanif040/kfc-static:21 -n default'
            }
        }
    }
}

}
    }

