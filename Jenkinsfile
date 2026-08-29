```groovy
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
                    sh '''
                        docker build -t devops-app:${BUILD_NUMBER} .
                    '''
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

                        docker tag devops-app:${BUILD_NUMBER} \
                        $DOCKERHUB_USERNAME/devops-app:${BUILD_NUMBER}

                        docker push \
                        $DOCKERHUB_USERNAME/devops-app:${BUILD_NUMBER}
                    '''
                }
            }
        }

        stage('Deploy DEV') {
            steps {
                sh '''
                    helm upgrade --install devops-app-dev helm/devops-app \
                    -f helm/devops-app/values-dev.yaml \
                    --set image.tag=${BUILD_NUMBER} \
                    --namespace dev \
                    --kube-insecure-skip-tls-verify
                '''
            }
        }

        stage('Deploy QA') {
            steps {
                sh '''
                    helm upgrade --install devops-app-qa helm/devops-app \
                    -f helm/devops-app/values-qa.yaml \
                    --set image.tag=${BUILD_NUMBER} \
                    --namespace qa \
                    --kube-insecure-skip-tls-verify
                '''
            }
        }

        stage('Deploy PROD') {
            steps {
                sh '''
                    helm upgrade --install devops-app-prod helm/devops-app \
                    -f helm/devops-app/values-prod.yaml \
                    --set image.tag=${BUILD_NUMBER} \
                    --namespace prod \
                    --kube-insecure-skip-tls-verify
                '''
            }
        }
    }
}
```
