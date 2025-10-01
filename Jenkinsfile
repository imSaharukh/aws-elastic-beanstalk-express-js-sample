pipeline {
    agent any  // Use Jenkins container itself

    environment {
        APP_NAME = "aws-elastic-beanstalk-express-js-sample"
        NODE_VERSION = "16"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/imSaharukh/aws-elastic-beanstalk-express-js-sample'
            }
        }

        stage('Install Dependencies') {
            steps {
                // Run npm install using the Jenkins container's Node 16 (must have Node installed)
                sh "npm install"
            }
        }

        stage('Run Tests') {
            steps {
                sh "npm test"
            }
        }

        stage('Security Scan') {
            steps {
                // Optional: Snyk scan if required for assignment
                sh "npx snyk test || true"
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs.'
        }
        always {
            cleanWs()
        }
    }
}
