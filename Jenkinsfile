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
            dockerImage.push("latest")
          }
        }
      }
    }


    stage('Sonarqube Analysis') {
      steps {
        script {
        def scannerHome = tool 'SonarqubeScanner-4.8.0';
        withSonarQubeEnv(credentialsId: 'sonarqubenew') {
          sh  "${tool("SonarqubeScanner-4.8.0")}/bin/sonar-scanner \
              -Dsonar.projectKey=nodeapp \
              -Dsonar.sources=. \
              -Dsonar.host.url=http://172.16.16.153:9000 \
              -Dsonar.login=sqa_d62b77bd555d96c472964e1b7ba4f825f9cb9126"
          }
        }
      }
    }

  }

}
