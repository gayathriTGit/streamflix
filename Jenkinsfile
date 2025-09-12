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

     stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'KUBECONFIG_STREAMFLIX', variable: 'KUBECONFIG')]) {
                  sh 'kubectl config view -o jsonpath="{.clusters[0].cluster.server}" && echo'
                  sh 'kubectl get nodes'
                }
                kubernetesDeploy(
                    kubeconfigId: 'KUBECONFIG_STREAMFLIX', // ID of your Kubernetes credentials in Jenkins                    
                    configs: 'kubernetes/*.yaml', // Path to your Kubernetes manifest files
                )
            }
        }

  }

}
