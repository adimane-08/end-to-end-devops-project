pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                dir('application') {
                    sh 'chmod +x mvnw'
                    sh './mvnw clean package'
                }
            }
        }

        stage('Test') {
            steps {
                dir('application') {
                    sh './mvnw test'
                }
            }
        }

        stage('Docker Build') {
            steps {
                dir('application') {
                    sh 'docker build -t devops-app:v2 .'
                }
            }
        }
        stage('Docker Push') {
          steps {
            withCredentials([usernamePassword(
              credentialsId: 'dockerhub-credentials',
              usernameVariable: 'DOCKERHUB_USERNAME',
              passwordVariable: 'DOCKERHUB_TOKEN'
        )]) {
            sh '''
                echo "$DOCKERHUB_TOKEN" | docker login -u "$DOCKERHUB_USERNAME" --password-stdin
                docker tag devops-app:v2 $DOCKERHUB_USERNAME/devops-app:v2
                docker push $DOCKERHUB_USERNAME/devops-app:v2
            '''
        }
    }
    }
}
}