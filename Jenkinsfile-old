pipeline {

  environment {
    dockerimagename = "sarfecosmob/nodeapp"
    dockerImage = ""
  }

  agent any

  stages {

    stage('Checkout Source') {
      steps {
        git 'https://github.com/sarfaraz-ecosmob/nodeapp.git'
      }
    }

    stage('Build image') {
      steps{
        script {
          dockerImage = docker.build dockerimagename
        }
      }
    }

    stage('Pushing Image') {
      environment {
               registryCredential = 'dockerhublogin'
           }
      steps{
        script {
          docker.withRegistry( 'https://registry.hub.docker.com', registryCredential ) {
            dockerImage.push("v1")
          }
        }
      }
    }


    stage('Sonarqube Analysis') {
      steps {
        script {
        def scannerHome = tool 'sonarscanner';
        withSonarQubeEnv(credentialsId: 'sonarqube') {
          sh  "${tool("sonarscanner")}/bin/sonar-scanner \
              -Dsonar.projectKey=nodeapp \
              -Dsonar.sources=. \
              -Dsonar.host.url=http://172.16.16.71:9000 \
              -Dsonar.login=sqa_4a07fa2168d3c8befd2044aa88aad9081f04729d"
          }
        }
      }
    }

  }

}
