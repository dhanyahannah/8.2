pipeline {
  agent any
  options { skipDefaultCheckout(true); timestamps() }

  // Polls Git every ~2 minutes so a README change will auto-trigger
  triggers { pollSCM('H/2 * * * *') }

  environment {
    EMAIL_TO    = 'mloganathan0701@gmail.com'          // <-- your email
    REPO_URL    = 'https://github.com/dhanyahannah/8.2.git' // <-- your repo URL if different
    REPO_BRANCH = 'main'
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: "${env.REPO_BRANCH}", url: "${env.REPO_URL}"
      }
    }

    stage('Env Info') {
      steps {
        // quick sanity check Node/NPM are available to Jenkins
        bat 'node -v & npm -v'
      }
    }

    stage('Install Dependencies') {
      steps {
        // fast when lockfile exists; falls back if needed
        bat 'npm ci --no-fund --no-audit || npm install'
      }
    }

    stage('Run Tests') {
      steps {
        // If tests fail, mark THIS STAGE failed but keep overall build SUCCESS so emails still send
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          bat 'npm test'
        }
      }
      post {
        success {
          emailext(
            to: "${env.EMAIL_TO}",
            subject: "[SUCCESS] ${env.JOB_NAME} #${env.BUILD_NUMBER} – Run Tests",
            body: "Run Tests stage succeeded for ${env.JOB_NAME} #${env.BUILD_NUMBER}.",
            attachLog: true
          )
        }
        failure {
          emailext(
            to: "${env.EMAIL_TO}",
            subject: "[FAILURE] ${env.JOB_NAME} #${env.BUILD_NUMBER} – Run Tests",
            body: "Run Tests stage FAILED for ${env.JOB_NAME} #${env.BUILD_NUMBER}. See attached log.",
            attachLog: true
          )
        }
      }
    }

    stage('Generate Coverage Report') {
      steps {
        // don’t fail if no coverage script exists
        bat 'npm run coverage || exit /b 0'
      }
    }

    stage('NPM Audit (Security Scan)') {
      steps {
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          bat 'npm audit'
        }
      }
      post {
        success {
          emailext(
            to: "${env.EMAIL_TO}",
            subject: "[SUCCESS] ${env.JOB_NAME} #${env.BUILD_NUMBER} – Security Scan",
            body: "Security Scan stage succeeded for ${env.JOB_NAME} #${env.BUILD_NUMBER}.",
            attachLog: true
          )
        }
        failure {
          emailext(
            to: "${env.EMAIL_TO}",
            subject: "[FAILURE] ${env.JOB_NAME} #${env.BUILD_NUMBER} – Security Scan",
            body: "Security Scan stage FAILED for ${env.JOB_NAME} #${env.BUILD_NUMBER}. See attached log.",
            attachLog: true
          )
        }
      }
    }
  }
}
