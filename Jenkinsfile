pipeline {

  agent any

  environment {
    quay-credentials=credentials('quay-credentials') // Create a credentials in jenkins using your dockerhub username and token from https://hub.docker.com/settings/security
  }


  stages {

    stage("Git Checkout") {
      steps {
        script {
           sh "git clone https://github.com/DashrathMundkar/cicd-java-maven-project.git"
        }
      }
    }

    stage("Maven Build") {
      steps {
        script {
          sh "mvn clean install -T 1C" // -T 1C is to make build faster using multithreading
        }
      }
    }

   stage("Run SonarQube Analysis") {
      steps {
        script {
          withSonarQubeEnv('YOUR_SonarQube_INSTALLATION_NAME') {
           sh 'mvn clean package sonar:sonar -Dsonar.profile="Sonar way"'
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
          sh "docker build -t dash18/cicd-java-maven ."
          sh "docker push dash18/cicd-java-maven"
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
