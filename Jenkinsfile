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
                        echo "$DOCKERHUB_TOKEN" | docker login \
                        -u "$DOCKERHUB_USERNAME" --password-stdin

                        docker tag devops-app:v2 \
                        $DOCKERHUB_USERNAME/devops-app:v2

                        docker push \
                        $DOCKERHUB_USERNAME/devops-app:v2
                    '''
                }
            }
        }

        stage('Deploy DEV') {
            steps {
                sh '''
                    helm upgrade --install devops-app-dev helm/devops-app \
                    -f helm/devops-app/values-dev.yaml \
                    --namespace dev
                '''
            }
        }

        stage('Deploy QA') {
            steps {
                sh '''
                    helm upgrade --install devops-app-qa helm/devops-app \
                    -f helm/devops-app/values-qa.yaml \
                    --namespace qa
                '''
            }
        }

        stage('Deploy PROD') {
            steps {
                sh '''
                    helm upgrade --install devops-app-prod helm/devops-app \
                    -f helm/devops-app/values-prod.yaml \
                    --namespace prod
                '''
            }
        }
    }
}