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
      
      stage('Trivy FS scan (repo)') {
        steps {
          // fails build on HIGH/CRITICAL; secret scan included by default
          sh '''
            trivy fs --no-progress \
              --severity CRITICAL,HIGH \
              --exit-code 1 \
              --format sarif -o trivy-fs.sarif .
          '''
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

      stage('Trivy scan image and Push Docker Image') {
        steps {
          sh '''
            trivy image --no-progress \
              --severity CRITICAL,HIGH --ignore-unfixed --exit-code 1 \
              --format sarif -o trivy-image.sarif "$IMAGE":${TAG}

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

      stage('OWASP baseline scan for http://13.222.10.181:30007/') {
        steps {
          sh '''
            # Run ZAP baseline w/ proper mount + uid/gid
            set -e
            ART="$WORKSPACE/zap_artifacts"
            mkdir -p "$ART"
            # Save reports under $WORKSPACE/zap_artifacts/*
            docker run --rm -u "$(id -u):$(id -g)" -v "$ART:/zap/wrk:rw" \
                zaproxy/zap-stable:latest \
                zap-baseline.py -m 5 \
                  -t http://13.222.10.181:30007/ \
                  -r zap_baseline_report.html || true
          '''
        }
      }

    }
    post {
        always {
            archiveArtifacts artifacts: 'trivy-*.sarif,trivy-*.txt,zap_artifacts/**', fingerprint: true
       }
    }
}
