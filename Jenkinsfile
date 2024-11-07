pipeline {
  agent any

  environment {
    DOCKER_IMAGE_NAME = "sarfecosmob/nodeapp"
    DOCKER_TAG = "build-${BUILD_NUMBER}" // Use BUILD_NUMBER for unique tagging
    REGISTRY_CREDENTIAL = 'dockerhublogin'
    SONAR_CREDENTIAL = 'sonarqube'
  }

  stages {

    stage('Checkout Source') {
      steps {
        git 'https://github.com/sarfaraz-ecosmob/nodeapp.git'
      }
    }

    stage('Build and Push Docker Image') {
      steps {
        script {
          // Authenticate, build, and push within the same stage
          docker.withRegistry('https://registry.hub.docker.com', REGISTRY_CREDENTIAL) {
            def dockerImage = docker.build("${DOCKER_IMAGE_NAME}:${DOCKER_TAG}")
            dockerImage.push(DOCKER_TAG) // Push with the unique tag
          }
        }
      }
    }

    stage('SonarQube Analysis') {
      steps {
        script {
          def scannerHome = tool 'sonarscanner'
          withSonarQubeEnv(credentialsId: SONAR_CREDENTIAL) {
            sh "${scannerHome}/bin/sonar-scanner \
                -Dsonar.projectKey=nodeapp \
                -Dsonar.sources=. \
                -Dsonar.host.url=http://172.16.16.71:9000 \
                -Dsonar.login=${SONAR_CREDENTIAL}"
          }
        }
      }
    }
  }
}

