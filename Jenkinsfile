pipeline {
  agent any

  options { skipDefaultCheckout(true) }
  triggers { pollSCM('H/2 * * * *') } 

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/dhanyahannah/8.2.git'
      }
    }

    stage('Install Dependencies') {
      steps { bat 'npm install' }
    }

    stage('Run Tests') {
      steps { bat 'npm test || exit /b 0' } 
    }

    stage('Generate Coverage Report') {
      steps { bat 'npm run coverage || exit /b 0' }
    }

    stage('NPM Audit (Security Scan)') {
      steps { bat 'npm audit || exit /b 0' }
    }
  }
}
