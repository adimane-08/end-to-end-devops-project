pipeline {
  agent any
  stages {
    
    stage('checkout') {
      steps { 
          checkout scm
          }
     }

    stage('build') {
      steps {
        dir('application') {
          bat '.\\mvnw.cmd clean package'
          }
        }
      }
    stage('test')
      steps {
        dir('application') {
            bat '.\\mvnw.cmd test'
          }
        }
      }
    }
  }

  