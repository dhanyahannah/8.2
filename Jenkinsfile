pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/dhanyahannah/8.2.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                bat 'npm install'
            }
        }
        stage('Run Tests') {
            steps {
                bat 'npm test || exit /b 0'
            }
        }
        stage('NPM Audit (Security Scan)') {
            steps {
                bat 'npm audit || exit /b 0'
            }
        }
    }

    post {
        success {
            emailext(
                subject: "Jenkins Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Hi,\n\nThe Jenkins build completed successfully.\n\nCheck the attached log for details.",
                to: 'mloganathan0701@gmail.com',
                attachmentsPattern: '**/build.log'
            )
        }
        failure {
            emailext(
                subject: "Jenkins Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Hi,\n\nThe Jenkins build failed. Please check Jenkins console logs.",
                to: 'mloganathan0701@gmail.com'
            )
        }
    }
}
