pipeline {
    agent {
        docker {
            image 'node:16'
            args '-u root:root' // run as root inside container
        }
    }

    environment {
        DOCKERHUB_USER = 'imsaharukh' 
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing npm dependencies...'
                sh 'npm install --save'
            }
        }

        stage('Run Tests') {
            steps {
                echo 'Running unit tests...'
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh 'docker build -t $DOCKERHUB_USER/aws-sample-app .'
            }
        }

        stage('Security Scan') {
            steps {
                echo 'Running Snyk vulnerability scan...'
                sh 'npm install -g snyk'
                sh 'snyk test || exit 1' 
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-token', 
                                                  usernameVariable: 'DOCKER_USER', 
                                                  passwordVariable: 'DOCKER_PASS')]) {
                    echo 'Logging into Docker Hub...'
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    echo 'Pushing Docker image to Docker Hub...'
                    sh 'docker push $DOCKER_USER/aws-sample-app'
                }
            }
        }

    }

    post {
        always {
            echo 'Cleaning workspace...'
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}
