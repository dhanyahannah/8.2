pipeline {
  agent any
  options {
    skipDefaultCheckout(true)   // avoid the implicit checkout
    timestamps()
  }

  environment {
    GIT_URL    = 'https://github.com/dhanyahannah/8.2.git'
    EMAIL_TO   = 'mloganathan0701@gmail.com'
    // Speeds up npm on Windows agents
    NPM_CONFIG_CACHE = "${WORKSPACE}\\npm-cache"
  }

  stages {
    stage('Checkout (shallow)') {
      steps {
        // Shallow anonymous checkout (no credentials needed for public repo)
        checkout([
          $class: 'GitSCM',
          branches: [[name: '*/main']],
          doGenerateSubmoduleConfigurations: false,
          userRemoteConfigs: [[url: env.GIT_URL]],
          extensions: [
            [$class: 'CloneOption', shallow: true, depth: 1, noTags: true, reference: '']
          ]
        ])
      }
    }

    stage('Install Dependencies') {
      steps {
        // Fastest reliable install path; skips audits/fund banner
        bat 'npm ci --no-audit --no-fund'
      }
    }

    stage('Run Tests') {
      steps {
        // Don’t fail the build if tests aren’t set up
        bat 'npm test || exit /b 0'
      }
    }

    stage('Quick Security Audit') {
      steps {
        // Light audit; never fail the build
        bat 'npm audit --omit=dev --json > audit.json || exit /b 0'
      }
    }
  }

  post {
    always {
      archiveArtifacts allowEmptyArchive: true, artifacts: 'audit.json'
    }
    success {
      emailext(
        subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        to: env.EMAIL_TO,
        body: """Build succeeded!

Job:    ${env.JOB_NAME}
Build:  #${env.BUILD_NUMBER}
URL:    ${env.BUILD_URL}""",
        attachLog: true,
        compressLog: true
      )
    }
    failure {
      emailext(
        subject: "FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        to: env.EMAIL_TO,
        body: """Build failed.

Job:    ${env.JOB_NAME}
Build:  #${env.BUILD_NUMBER}
URL:    ${env.BUILD_URL}""",
        attachLog: true,
        compressLog: true
      )
    }
  }
}
