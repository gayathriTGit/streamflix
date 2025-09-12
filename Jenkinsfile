pipeline {
    agent any
 environment {
    IMAGE = "gayathritdocker/streamflix-deployment"
    TAG   = "1.0" // or "${env.BUILD_NUMBER}"
  }
  stages {

    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/gayathriTGit/streamflix.git'
      }
    }

    stage('Docker login') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'DOCKER_HUB_CREDENTIALS',
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_TOKEN'
        )]) {
          sh '''
            echo "$DOCKER_TOKEN" | docker login -u "$DOCKER_USER" --password-stdin
          '''
        }
      }
    }

    stage('Build') {
      steps {
        sh '''
          docker version
          # docker rmi ${IMAGE}:${TAG} -f || true
          docker build -t ${IMAGE}:${TAG} .
        '''
      }
    }

    stage('Push Docker Image') {
      steps {
        sh '''
          docker push ${IMAGE}:${TAG}
        '''
      }
    }
}
