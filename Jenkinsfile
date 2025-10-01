pipeline {
    // Use any available agent (Jenkins container itself)
    agent any

    environment {
        // Set environment variables for Docker
        APP_NAME = "aws-elastic-beanstalk-express-js-sample"
        DOCKER_IMAGE = "node:16"
    }

    stages {

        stage('Checkout') {
            steps {
                // Checkout your forked repository from GitHub
                git branch: 'main', url: 'https://github.com/imSaharukh/aws-elastic-beanstalk-express-js-sample'
            }
        }

        stage('Install Dependencies') {
            steps {
                // Run npm install inside Node 16 Docker container
                sh """
                docker run --rm -v \$PWD:/app -w /app ${DOCKER_IMAGE} npm install --save
                """
            }
        }

        stage('Run Tests') {
            steps {
                // Run npm test inside Node 16 Docker container
                sh """
                docker run --rm -v \$PWD:/app -w /app ${DOCKER_IMAGE} npm test
                """
            }
        }

        stage('Build Docker Image') {
            steps {
                // Build the Docker image for the app
                sh "docker build -t ${APP_NAME}:latest ."
            }
        }

        stage('Security Scan') {
            steps {
                // Run Snyk security scan (requires SNYK_TOKEN in Jenkins credentials)
                sh """
                docker run --rm -v \$PWD:/app -w /app -e SNYK_TOKEN=${SNYK_TOKEN} snyk/snyk-cli test --all-projects
                """
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
        always {
            // Clean workspace after build
            cleanWs()
        }
    }
}
