pipeline {

  environment {
    dockerimagename = "sarfecosmob/nodeapp"
    dockerImage = ""
    versionFile = "version.txt"
  }

  agent any

  stages {

    stage('Checkout Source') {
      steps {
        git 'https://github.com/sarfaraz-ecosmob/nodeapp.git'
      }
    }

    stage('Get Latest Version') {
      steps {
        script {
          // Fetch or read the current version number, increment it, and set `newVersion`
          def currentVersion = readFile(versionFile).trim()
          def versionNumber = currentVersion.replace("v", "").toInteger() + 1
          newVersion = "v" + versionNumber
          echo "New version: ${newVersion}"
        }
      }
    }

    stage('Build Image') {
      steps {
        script {
          dockerImage = docker.build("${dockerimagename}:${newVersion}")
        }
      }
    }

    stage('Push Image') {
      environment {
        registryCredential = 'dockerhublogin'
      }
      steps {
        script {
          docker.withRegistry('https://registry.hub.docker.com', registryCredential) {
            dockerImage.push(newVersion)
          }
        }
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

    stage('Update Version File') {
      steps {
        script {
          // Update version file and commit changes
          writeFile file: versionFile, text: newVersion
          sh 'git config user.name "Jenkins"'
          sh 'git config user.email "jenkins@example.com"'
          sh "git add ${versionFile}"
          sh 'git commit -m "Increment version to ${newVersion}"'
          sh 'git push origin main'  // Update this to the correct branch if needed
        }
      }
    }

  }
}

