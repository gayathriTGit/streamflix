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
            docker build --no-cache --build-arg TMDB_V3_API_KEY=ac3b6efa58f183f1a4ed954f10af69ba -t ${IMAGE}:${TAG} .
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

     stage('Deploy to Kubernetes') {
        steps {
            withCredentials([file(credentialsId: 'KUBECONFIG_STREAMFLIX', variable: 'KUBECONFIG')]) {
              sh 'kubectl config view -o jsonpath="{.clusters[0].cluster.server}" && echo'
              sh 'kubectl get nodes'
              sh 'kubectl apply -f Kubernetes/deployment.yml'
              sh 'kubectl apply -f Kubernetes/service.yml'
            }                
        }
     }

  }

}
