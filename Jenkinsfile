pipeline {
    agent any
    environment {
            // 'dockerhub-creds' is the ID you assigned to your credentials in Jenkins
            DOCKER_HUB_CREDS = credentials('DOCKER_HUB_CREDENTIALS')
        }
    stages {
     
        stage('Checkout') {
            steps { 
                script {
                    git branch: 'main', url: 'https://github.com/gayathriTGit/streamflix.git'
                }
            }
        }

        stage('Build') {
            steps {
                sh '''
                docker version
                // docker rmi gayathritdocker/streamflix-deployment:1.0 -f || true 
				sudo docker build -t streamflix-deployment:1.0 .
		        sudo docker tag streamflix-deployment:1.0 gayathritdocker/streamflix-deployment:1.0
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'DOCKER_HUB_CREDENTIALS', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "docker login -u ${DOCKER_USER} -p ${DOCKER_PASS}"
                    sh "docker push gayathritdocker/streamflix-deployment:1.0"
                    sh "docker logout" // Optional: log out after pushing
                }
            }
        }
    
        stage('Deploy to Kubernetes') {
            steps {
                script {
                   // Example using kubectl
                   sh 'kubectl apply -f deployment.yaml'
                   sh 'kubectl apply -f service.yaml'
                }
            }
       }
    }
}
