pipeline {

  agent any
        tools {
        maven "Maven"
        }
  environment {
    quay-credentials=credentials('quay-credentials') // Create a credentials in jenkins using your dockerhub username and token from https://hub.docker.com/settings/security
  }


 stages {
        stage('Checkout') {
            steps {
                git branch: 'maheshnemade98-patch-1', url: 'https://github.com/maheshnemade98/cicd-java-maven-project.git', credentialsId: "github"
            }
        }

    stage("Maven Build") {
      steps {
        script {
          sh "mvn clean install -T 1C" // -T 1C is to make build faster using multithreading
        }
      }
    }

   stage('SonarQube Scanner') {
            steps {
                script {
                    def sonarScannerHome = tool name: 'sonar-scanner'
                    withSonarQubeEnv('sonar-server') {
                        sh """
                        ${sonarScannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=sonar \
                        -Dsonar.projectName=sonar \
                        -Dsonar.projectSources=. \
                        """
                    }
                }
            }
        }

          try {
            timeout(time: 5, unit: 'MINUTES') { // pipeline will be killed after a timeout
              def qg = waitForQualityGate()
              if (qg.status != 'OK') {
                error "Pipeline aborted due to quality gate failure: ${qg.status}"
              }
            }
          } catch (e) {
            throw e
          }
        }
      }
    }

    stage("Build & Push Docker Image") {
      steps {
        script {
          sh 'echo $quay-credentials_PSW | docker login -u $quay-credentials_USR --password-stdin'
          sh "docker build -t quay.io/mahesh_nemade/cicd-java-maven ."
          sh "docker push quay.io/mahesh_nemade/cicd-java-maven"
        }
      }
    }

    stage("Apply the Kubernetes files") {
      steps {
        script {
          sh "kubectl apply -f kubernetes/ "
        }
      }
    }
  }
  post {
    always {
      script {
        if (currentBuild.currentResult == 'FAILURE') {
          step([$class: 'Mailer', notifyEveryUnstableBuild: true, recipients: "Test@test.com", sendToIndividuals: true])
        }
      }
    }
  }
}
