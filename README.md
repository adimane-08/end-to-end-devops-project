\# End-to-End DevOps CI/CD Project



\## 📌 Project Overview



This project demonstrates an end-to-end DevOps CI/CD implementation for a Spring Boot application.



The project automates the application lifecycle from source code checkout through build, testing, containerization, Docker image publishing, and Kubernetes deployment across DEV, QA, and PROD environments.



The pipeline follows a \*\*Build Once, Promote the Same Artifact\*\* approach using a Jenkins build number as the Docker image tag.



\---



\## 🏗️ Architecture



```text

Developer

&#x20;   |

&#x20;   | Git Push

&#x20;   v

+----------------+

|     GitHub     |

| Source Code +  |

|  Jenkinsfile   |

+-------+--------+

&#x20;       |

&#x20;       v

+-------------------------+

|        Jenkins          |

|                         |

|  1. Checkout            |

|  2. Maven Build         |

|  3. Maven Test          |

|  4. Docker Build        |

|  5. Docker Push         |

|  6. Helm Deployment     |

+-----------+-------------+

&#x20;           |

&#x20;           | Docker Image

&#x20;           | :BUILD\_NUMBER

&#x20;           v

+-------------------------+

|       Docker Hub        |

|                         |

| adimane0801/devops-app  |

|          :17            |

+-----------+-------------+

&#x20;           |

&#x20;           v

+----------------------------------+

|       Kubernetes / Minikube      |

|                                  |

|   DEV                            |

|     |                            |

|     | Manual Approval            |

|     v                            |

|   QA                             |

|     |                            |

|     | Manual Approval            |

|     v                            |

|   PROD                           |

+----------------------------------+

```



\---



\## 🛠️ Technology Stack



| Technology           | Purpose                                  |

| -------------------- | ---------------------------------------- |

| GitHub               | Source code management                   |

| Jenkins              | CI/CD automation                         |

| Maven                | Build and test automation                |

| Docker               | Application containerization             |

| Docker Hub           | Container image registry                 |

| Kubernetes           | Container orchestration                  |

| Minikube             | Local Kubernetes environment             |

| Helm                 | Kubernetes package/deployment management |

| Spring Boot Actuator | Application health monitoring            |

| Linux                | Build/deployment environment             |



\---



\## 🔄 CI/CD Pipeline



The Jenkins pipeline performs the following stages:



\### 1. Checkout



Jenkins checks out the application source code from GitHub.



```groovy

checkout scm

```



\### 2. Build



The Spring Boot application is built using Maven:



```bash

./mvnw clean package

```



\### 3. Test



Automated tests are executed:



```bash

./mvnw test

```



\### 4. Docker Build



A Docker image is created using the Jenkins build number:



```bash

docker build -t devops-app:${BUILD\_NUMBER} .

```



For example:



```text

Jenkins Build #17

&#x20;       ↓

devops-app:17

```



\### 5. Docker Push



The image is tagged with the Docker Hub username:



```bash

docker tag devops-app:${BUILD\_NUMBER} \\

$DOCKERHUB\_USERNAME/devops-app:${BUILD\_NUMBER}

```



The image is then pushed:



```bash

docker push \\

$DOCKERHUB\_USERNAME/devops-app:${BUILD\_NUMBER}

```



Example:



```text

adimane0801/devops-app:17

```



\---



\## 🐳 Docker Image Versioning



The pipeline uses the Jenkins `BUILD\_NUMBER` as the Docker image tag.



For example:



```text

Build #14 → devops-app:14

Build #15 → devops-app:15

Build #16 → devops-app:16

Build #17 → devops-app:17

```



This provides traceability between:



```text

Jenkins Build

&#x20;     ↓

Docker Image

&#x20;     ↓

Kubernetes Deployment

```



For Build #17:



```text

Jenkins Build: 17

Docker Image: adimane0801/devops-app:17

```



\---



\## ☸️ Kubernetes Deployment



Helm is used to deploy the application to Kubernetes.



Separate values files are maintained for each environment:



```text

helm/devops-app/

├── Chart.yaml

├── values.yaml

├── values-dev.yaml

├── values-qa.yaml

├── values-prod.yaml

└── templates/

```



The deployment uses the image tag supplied by Jenkins:



```bash

\--set image.tag=${BUILD\_NUMBER}

```



Therefore Build #17 deploys:



```text

adimane0801/devops-app:17

```



\---



\## 🌎 Environment Promotion



The pipeline promotes the same Docker image through three environments.



```text

&#x20;             Docker Image :17

&#x20;                    |

&#x20;                    v

&#x20;                 DEV :17

&#x20;                    |

&#x20;              Approval

&#x20;                    |

&#x20;                    v

&#x20;                  QA :17

&#x20;                    |

&#x20;              Approval

&#x20;                    |

&#x20;                    v

&#x20;                 PROD :17

```



\### DEV



```bash

helm upgrade --install devops-app-dev \\

helm/devops-app \\

\-f helm/devops-app/values-dev.yaml \\

\--set image.tag=${BUILD\_NUMBER} \\

\--namespace dev

```



\### QA



```bash

helm upgrade --install devops-app-qa \\

helm/devops-app \\

\-f helm/devops-app/values-qa.yaml \\

\--set image.tag=${BUILD\_NUMBER} \\

\--namespace qa

```



\### PROD



```bash

helm upgrade --install devops-app-prod \\

helm/devops-app \\

\-f helm/devops-app/values-prod.yaml \\

\--set image.tag=${BUILD\_NUMBER} \\

\--namespace prod

```



\---



\## 👤 Manual Approval Gates



The pipeline requires approval before promoting the application from DEV to QA and from QA to PROD.



\### DEV → QA



```groovy

input message: 'DEV deployment completed. Deploy to QA?', ok: 'Deploy QA'

```



\### QA → PROD



```groovy

input message: 'QA deployment completed. Deploy to PROD?', ok: 'Deploy PROD'

```



This provides controlled environment promotion.



\---



\## ❤️ Application Health Validation



The Spring Boot Actuator health endpoint is used to verify application health.



```bash

curl http://localhost:8081/actuator/health

```



Example response:



```json

{

&#x20; "groups": \[

&#x20;   "liveness",

&#x20;   "readiness"

&#x20; ],

&#x20; "status": "UP"

}

```



This confirms that the application is running and reporting healthy liveness/readiness status.



\---



\## 🔍 Kubernetes Validation



The application deployment can be verified using:



```bash

kubectl get pods -n dev

```



```bash

kubectl get pods -n qa

```



```bash

kubectl get pods -n prod

```



Deployment rollout can be checked using:



```bash

kubectl rollout status deployment/devops-app-dev -n dev

```



The deployed image can be verified using:



```bash

kubectl get deployment devops-app-prod \\

\-n prod \\

\-o jsonpath="{.spec.template.spec.containers\[0].image}"

```



\---



\## 🧪 Example Successful Deployment



A successful production deployment produces:



```text

Release "devops-app-prod" has been upgraded.

STATUS: deployed

```



Example:



```text

Jenkins Build: 17

Docker Image: adimane0801/devops-app:17

Environment: PROD

Helm Release: devops-app-prod

Status: deployed

```



\---



\## 🛠️ Troubleshooting Performed



During implementation, several real-world DevOps issues were identified and resolved.



\### Docker Socket Permission



Jenkins initially could not access Docker:



```text

permission denied while trying to connect to the Docker daemon socket

```



The Jenkins container was configured to access:



```text

/var/run/docker.sock

```



and the Jenkins user was given appropriate group access.



\### Kubernetes Connectivity



Jenkins was configured to access the Kubernetes cluster using a kubeconfig mounted into the container.



\### Jenkins/Git Connectivity



Git connectivity and DNS resolution from the Jenkins container were validated before continuing with the pipeline.



\### Helm Deployment



Environment-specific Helm values files were used to deploy the application to separate namespaces.



\### Application Validation



The application was validated through:



```text

/actuator/health

```



before promotion through the environments.



\---



\## 📁 Project Structure



```text

end-to-end-devops-project/

│

├── application/

│   ├── src/

│   ├── pom.xml

│   ├── mvnw

│   └── Dockerfile

│

├── helm/

│   └── devops-app/

│       ├── Chart.yaml

│       ├── values.yaml

│       ├── values-dev.yaml

│       ├── values-qa.yaml

│       ├── values-prod.yaml

│       └── templates/

│

├── Jenkinsfile

├── .gitignore

└── README.md

```



> Sensitive local files such as kubeconfig files should not be committed to Git.



\---



\## 🎯 Key DevOps Concepts Demonstrated



\* Continuous Integration

\* Continuous Delivery

\* Jenkins Declarative Pipeline

\* Git-based source control

\* Maven build automation

\* Automated testing

\* Docker containerization

\* Docker image versioning

\* Docker Hub image publishing

\* Kubernetes deployments

\* Kubernetes namespaces

\* Helm charts

\* Environment-specific Helm values

\* Manual deployment approvals

\* Build-once/promote-artifact strategy

\* Application health checks

\* Kubernetes rollout validation

\* CI/CD troubleshooting



\---



\## 💡 Interview Explanation



A concise way to explain this project:



> I implemented an end-to-end Jenkins CI/CD pipeline for a Spring Boot application. The pipeline checks out the code from GitHub, builds and tests the application using Maven, creates a Docker image using the Jenkins build number as the tag, and pushes the image to Docker Hub. Helm is then used to deploy the same image across DEV, QA, and PROD Kubernetes namespaces. Manual approval gates are implemented between environments, and Spring Boot Actuator is used to validate application health before promotion. This follows a build-once, promote-the-same-artifact approach.



\---



\## 🚀 Future Improvements



Possible enhancements include:



\* Automated DEV health checks inside Jenkins

\* Automated QA test execution

\* Automated rollback on deployment failure

\* Kubernetes Horizontal Pod Autoscaling

\* Prometheus and Grafana monitoring

\* SonarQube code-quality analysis

\* Security scanning

\* Slack/email deployment notifications

\* GitHub webhook-based automatic pipeline triggering

\* Blue/Green or Canary deployments

\* Infrastructure provisioning using Terraform



```



\*\*Important:\*\* Before publishing this README, remove or replace any details that don't exactly match your final repository. In particular, keep `kubeconfig` out of Git and make sure `.gitignore` excludes it.



\### Your project is now ready for the next evidence layer



I would next create \*\*3 strong resume bullets + a LinkedIn project post\*\*, both based only on what you've actually implemented.



Which next: \*\*resume bullets\*\* or \*\*LinkedIn post\*\*?

```



