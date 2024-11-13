pipeline {
  agent any

  environment {
    DOCKER_IMAGE_NAME = "sarfecosmob/nodeapp"
    DOCKER_TAG = "build-${BUILD_NUMBER}" // Use BUILD_NUMBER for unique tagging
    REGISTRY_CREDENTIAL = 'dockerhublogin'
    SONAR_CREDENTIAL = 'sonarqube'
    GIT_REPO = 'https://github.com/your-username/argo.git' // Update with your GitHub username
  }

  stages {

    stage('Checkout Source') {
      steps {
        git 'https://github.com/sarfaraz-ecosmob/nodeapp.git'
      }
    }

    stage('Sonarqube Analysis') {
      steps {
        script {
          def scannerHome = tool 'sonarscanner'
          withSonarQubeEnv(credentialsId: 'sonarqube') {
            sh "${scannerHome}/bin/sonar-scanner \
              -Dsonar.projectKey=nodeapp \
              -Dsonar.sources=. \
              -Dsonar.host.url=http://172.16.16.71:9000 \
              -Dsonar.login=sqa_4a07fa2168d3c8befd2044aa88aad9081f04729d"
          }
        }
      }
    }

    stage('Build and Push Docker Image') {
      steps {
        script {
          docker.withRegistry('https://registry.hub.docker.com', REGISTRY_CREDENTIAL) {
            def dockerImage = docker.build("${DOCKER_IMAGE_NAME}:${DOCKER_TAG}")
            dockerImage.push(DOCKER_TAG)
          }
        }
      }
    }

    stage('Wait') {
            steps {
                echo 'Waiting for 30 seconds...'
                sleep time: 30, unit: 'SECONDS'
            }
        }


    stage('Update Manifest') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'GitHubCredentials', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')])
         {
          sh "rm -rf argocd"
          sh "git clone https://github.com/sarfaraz-ecosmob/argocd.git"
          dir('argocd') {
            sh "sed -i \"s|image: sarfecosmob/nodeapp:build-[0-9]*|image: sarfecosmob/nodeapp:${DOCKER_TAG}|g\" k8s/deploymentservice.yml"
            sh "git config user.email 'sarfaraz.shaikh@ecosmob.com'"
            sh "git config user.name 'sarfaraz-ecosmob'"
            sh "git add -A ."
            sh "git commit -m 'Update image version to: v${DOCKER_TAG}'"
            sh "git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/sarfaraz-ecosmob/argocd.git HEAD:main -f"
          }
        }
      }
    }
  }
}

