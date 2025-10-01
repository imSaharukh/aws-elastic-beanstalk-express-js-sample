pipeline {
    agent any

    environment {
        APP_DIR = "${WORKSPACE}"
        NODE_IMAGE = "node:16"
    }

    stages {

        stage('Checkout SCM') {
            steps {
                checkout([$class: 'GitSCM', 
                    branches: [[name: '*/main']], 
                    userRemoteConfigs: [[url: 'https://github.com/imSaharukh/aws-elastic-beanstalk-express-js-sample']]
                ])
            }
        }

        stage('Install Dependencies') {
            steps {
                sh """
                docker run --rm -v $APP_DIR:/app -w /app $NODE_IMAGE npm install
                """
            }
        }

        stage('Run Tests') {
            steps {
                sh """
                docker run --rm -v $APP_DIR:/app -w /app $NODE_IMAGE npm test
                """
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t my-app:latest $APP_DIR
                """
            }
        }

        stage('Security Scan') {
            steps {
                sh """
                echo "Security scan placeholder – integrate tools like Trivy or Snyk here"
                """
            }
        }
    }

    post {
        always {
            echo "Cleaning workspace..."
            cleanWs()
        }
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed. Check logs for details."
        }
    }
}
