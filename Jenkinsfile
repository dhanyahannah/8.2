pipeline {
  agent any
  options {
    skipDefaultCheckout(true)
    timestamps()
  }
  environment {
    GIT_URL  = 'https://github.com/dhanyahannah/8.2.git'
    EMAIL_TO = 'mloganathan0701@gmail.com'
    NPM_CONFIG_CACHE = "${WORKSPACE}\\npm-cache"
  }

  stages {
    stage('Checkout (shallow)') {
      steps {
        checkout([
          $class: 'GitSCM',
          branches: [[name: '*/main']],
          userRemoteConfigs: [[url: env.GIT_URL]],
          extensions: [[$class: 'CloneOption', shallow: true, depth: 1, noTags: true]]
        ])
      }
    }

    stage('Install Dependencies') {
      steps {
        // If your repo lacks package-lock.json, switch to: npm install --no-audit --no-fund
        bat 'npm ci --no-audit --no-fund'
      }
    }

    stage('Run Tests') {
      steps {
        script {
          // capture exit code but don't fail pipeline
          def rc = bat(script: 'npm test', returnStatus: true)
          env.TEST_STATUS = (rc == 0 ? 'SUCCESS' : "FAILURE (${rc})")
        }
      }
      post {
        always {
          emailext(
            subject: "Run Tests - ${env.TEST_STATUS}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            to: env.EMAIL_TO,
            from: 'mloganathan0701@gmail.com',
            mimeType: 'text/html',
            body: """
              <p>Stage <b>Run Tests</b> finished with: <b>${env.TEST_STATUS}</b>.</p>
              <p><a href="${env.BUILD_URL}console">Console log</a></p>
            """,
            attachLog: true,
            compressLog: true
          )
        }
      }
    }

    stage('Quick Security Audit') {
      steps {
        script {
          // write JSON report and capture exit code (npm audit often returns non-zero)
          def rc = bat(script: 'npm audit --omit=dev --json > audit.json', returnStatus: true)
          env.AUDIT_STATUS = (rc == 0 ? 'SUCCESS' : "FAILURE (${rc})")
        }
      }
      post {
        always {
          emailext(
            subject: "Security Audit - ${env.AUDIT_STATUS}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            to: env.EMAIL_TO,
            from: 'mloganathan0701@gmail.com',
            mimeType: 'text/html',
            body: """
              <p>Stage <b>Security Audit</b> finished with: <b>${env.AUDIT_STATUS}</b>.</p>
              <p>Attached: <code>audit.json</code>. Full log zipped.</p>
              <p><a href="${env.BUILD_URL}artifact/audit.json">audit.json</a></p>
            """,
            attachmentsPattern: 'audit.json',
            attachLog: true,
            compressLog: true
          )
        }
      }
    }
  }

  // Optional: a single wrap-up email for the whole build
  post {
    always {
      archiveArtifacts allowEmptyArchive: true, artifacts: 'audit.json'
      emailext(
        subject: "Build ${currentBuild.currentResult}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        to: env.EMAIL_TO,
        from: 'mloganathan0701@gmail.com',
        body: """Build result: ${currentBuild.currentResult}
Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
URL: ${env.BUILD_URL}""",
        attachLog: true,
        compressLog: true
      )
    }
  }
}

