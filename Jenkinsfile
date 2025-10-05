// Declarative Jenkins Pipeline
// Trigger: started by timer (cron)
// - Checks out the repository
// - Builds the Maven project
// - Runs tests and publishes JUnit results
// - Archives the produced JAR

pipeline {
  agent any

  // Run once a day at a randomized hour/minute: 'H H * * *'
  // Adjust the cron expression as needed (Jenkins uses 5-field cron)
  triggers {
    cron('H H * * *')
  }

  environment {
    // If your Jenkins has a specific Maven tool configured, replace this
    // value with the tool name and use the 'tool' step instead.
    MVN = 'mvn'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
        echo "Started by timer - checked out: ${env.GIT_COMMIT ?: 'unknown commit'}"
      }
    }

    stage('Build') {
      steps {
        script {
          if (isUnix()) {
            sh "${env.MVN} -B -DskipTests clean package"
          } else {
            bat "${env.MVN} -B -DskipTests clean package"
          }
        }
      }
    }

    stage('Test') {
      steps {
        script {
          if (isUnix()) {
            sh "${env.MVN} -B test"
          } else {
            bat "${env.MVN} -B test"
          }
        }
        // Publish test results
        junit '**/target/surefire-reports/*.xml'
      }
    }

    stage('Archive') {
      steps {
        archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
      }
    }
  }

  post {
    success {
      echo "Build succeeded: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
    }
    failure {
      echo "Build failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
    }
    always {
      echo "Finished: ${currentBuild.currentResult}"
    }
  }
}
