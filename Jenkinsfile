pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'node:16'
        WORKSPACE_DIR = "${env.WORKSPACE}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'docker run --rm -v $WORKSPACE_DIR:/app -w /app node:16 npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'docker run --rm -v $WORKSPACE_DIR:/app -w /app node:16 npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t my-app:latest $WORKSPACE_DIR'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([string(credentialsId: 'DOCKERHUB_REPO_PSW', variable: 'DOCKERHUB_PASSWORD')]) {
                    sh '''
                        echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_REPO --password-stdin
                        docker push my-app:latest
                    '''
                }
            }
        }

        stage('Snyk Security Scan') {
            steps {
                withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
                    sh 'docker run --rm -v $WORKSPACE_DIR:/app -w /app node:16 npx snyk test'
                }
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: '**/build/**', fingerprint: true
            }
        }
    }
}
