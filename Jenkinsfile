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
        stage('DEV Health Check') {
          steps {
            sh '''
            		echo "Waiting for DEV rollout..."

              kubectl --insecure-skip-tls-verify rollout status \
                deployment/devops-app-dev \
                -n dev \
                --timeout=120s

              echo "Checking DEV application health..."

              kubectl --insecure-skip-tls-verify delete pod dev-health-check \
               -n dev \
               --ignore-not-found

              kubectl --insecure-skip-tls-verify run dev-health-check \
                --restart=Never \
                --image=curlimages/curl:8.10.1 \
                -n dev \
                --command -- \
                curl -f --connect-timeout 10 \
                http://devops-app-dev-service/actuator/health

              kubectl --insecure-skip-tls-verify wait \
                --for=jsonpath='{.status.phase}'=Succeeded \
                pod/dev-health-check \
                -n dev \
                --timeout=120s

              kubectl --insecure-skip-tls-verify logs \
                dev-health-check \
                -n dev

            kubectl --insecure-skip-tls-verify delete pod \
                dev-health-check \
                -n dev \
                --ignore-not-found
        '''
    }
}
	stage('Approve QA') {
          steps {
            input message: 'DEV deployment completed. Deploy to QA?', ok: 'Deploy QA'
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
        stage('QA Health Check') {
         steps {
          sh '''
            echo "Waiting for QA rollout..."

            kubectl --insecure-skip-tls-verify rollout status \
                deployment/devops-app-qa \
                -n qa \
                --timeout=120s

            echo "Checking QA application health..."

            kubectl --insecure-skip-tls-verify delete pod qa-health-check \
                -n qa \
                --ignore-not-found

            kubectl --insecure-skip-tls-verify run qa-health-check \
                --restart=Never \
                --image=curlimages/curl:8.10.1 \
                -n qa \
                --command -- \
                curl -f http://devops-app-qa-service/actuator/health

            kubectl --insecure-skip-tls-verify wait \
                --for=jsonpath='{.status.phase}'=Succeeded \
                pod/qa-health-check \
                -n qa \
                --timeout=60s

            kubectl --insecure-skip-tls-verify logs \
                qa-health-check \
                -n qa

            kubectl --insecure-skip-tls-verify delete pod \
                qa-health-check \
                -n qa \
                --ignore-not-found
        '''
    }
}
       
        stage('Approve PROD') {
         steps {
          input message: 'QA deployment completed. Deploy to PROD?', ok: 'Deploy PROD'
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