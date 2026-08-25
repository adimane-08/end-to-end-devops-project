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
    }
}