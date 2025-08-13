pipeline {
  agent any

  environment {
    DOCKERHUB_USER = 'hanif040'
    IMAGE_NAME     = 'kfc-static'
    IMAGE_TAG      = "${env.BUILD_NUMBER}"                  // unique tag each build
    IMAGE_FULL     = "${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
    IMAGE_LATEST   = "${DOCKERHUB_USER}/${IMAGE_NAME}:latest"
    KUBECONFIG     = '/var/lib/jenkins/.kube/config'        // jenkins kubeconfig path
  }

  options {
    timestamps()
    ansiColor('xterm')
  }

  stages {
    stage('Checkout') {
      steps {
        // If your repo branch is master, change 'main' to 'master'
        git url: 'https://github.com/mhanif45/KFC.git', branch: 'main'
      }
    }

    stage('Docker Build') {
      steps {
        sh """
          set -e
          echo "🔨 Building Docker image..."
          docker build -t ${IMAGE_FULL} -t ${IMAGE_LATEST} .
        """
      }
    }

    stage('Docker Login & Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'docker-hub',
                                          usernameVariable: 'DH_USER',
                                          passwordVariable: 'DH_PASS')]) {
          sh """
            set -e
            echo "🔑 Logging into Docker Hub..."
            echo "\$DH_PASS" | docker login -u "\$DH_USER" --password-stdin
            echo "📤 Pushing image to Docker Hub..."
            docker push ${IMAGE_FULL}
            docker push ${IMAGE_LATEST}
            docker logout || true
          """
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        sh """
          set -e
          echo "🚀 Updating Kubernetes deployment..."

          # Find first container name in deployment
          CONTAINER=\$(kubectl get deployment kfc-static-deployment \
            -o jsonpath='{.spec.template.spec.containers[0].name}')

          # Update image to new tag
          kubectl set image deployment/kfc-static-deployment \\
            \${CONTAINER}=${IMAGE_FULL} --record

          # Wait for pods to be ready
          kubectl rollout status deployment/kfc-static-deployment --timeout=180s

          # Show final deployed image
          echo "✅ Now running imag

