pipeline {        // lowercase 'pipeline'
    agent any
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/mhanif45/KFC.git'
            }
        }

stage('Docker Build') {
    steps {
        script {
            docker.build("hanif040/kfc-static:${env.BUILD_NUMBER}")
        }
    }
}
    }
}

