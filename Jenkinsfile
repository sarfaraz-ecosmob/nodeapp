pipeline {
  agent any

  environment {
    DOCKER_IMAGE_NAME = "sarfecosmob/nodeapp"
    REGISTRY_CREDENTIAL = 'dockerhublogin'
    SONAR_CREDENTIAL = 'sonarqube'
  }

  stages {

    stage('Checkout Source') {
      steps {
        git 'https://github.com/sarfaraz-ecosmob/nodeapp.git'
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          // Use the Jenkins BUILD_NUMBER as the version tag for the Docker image
          env.DOCKER_TAG = "build-${BUILD_NUMBER}"
          env.DOCKER_IMAGE = docker.build("${DOCKER_IMAGE_NAME}:${DOCKER_TAG}")
          echo "Built Docker image: ${DOCKER_IMAGE_NAME}:${DOCKER_TAG}"
        }
      }
    }

    stage('Push Docker Image') {
      steps {
        script {
          docker.withRegistry('https://registry.hub.docker.com', REGISTRY_CREDENTIAL) {
            env.DOCKER_IMAGE.push(env.DOCKER_TAG)
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

