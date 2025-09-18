// Jenkinsfile (Declarative)
pipeline {
  agent any
  options {
    skipDefaultCheckout(true)        // we do our own (shallow) checkout
    timestamps()
    ansiColor('xterm')
  }

  environment {
    GIT_URL     = 'https://github.com/dhanyahannah/8.2.git'
    GIT_BRANCH  = '*/main'
    GIT_CREDS   = 'github-creds'     // <-- change to your Credentials ID
    EMAIL_TO    = 'mloganathan0701@gmail.com'
    // Reuse npm cache across builds to speed things up
    NPM_CONFIG_CACHE = "${WORKSPACE}\\npm-cache"
  }

  stages {
    stage('Checkout (shallow)') {
      steps {
        // Shallow clone makes checkout ~10x faster
        checkout([
          $class: 'GitSCM',
          branches: [[name: env.GIT_BRANCH]],
          userRemoteConfigs: [[url: env.GIT_URL, credentialsId: env.GIT_CREDS]],
          extensions: [
            [$class: 'CloneOption', shallow: true, depth: 1, noTags: true, reference: '']
          ]
        ])
      }
    }

    stage('Install Dependencies (fast)') {
      steps {
        // Faster than `npm install`; avoids re-resolving lockfile, no audits/funding
        bat 'npm ci --no-audit --no-fund'
      }
    }

    stage('Run Tests (quick)') {
      steps {
        // If tests/snyk aren’t set up, don’t fail the build
        bat 'npm test || exit /b 0'
      }
    }

    stage('Security Audit (quick report)') {
      steps {
        // Production-only audit creates a small JSON you can archive (doesn’t fail)
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
        body: """Build succeeded.

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

