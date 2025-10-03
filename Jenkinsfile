pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'node:16'
        DOCKERHUB_REPO = credentials('dockerhub-repo-name')
        SNYK_TOKEN = credentials('snyk-token')
        DOCKER_HOST = 'tcp://docker:2376'
        DOCKER_CERT_PATH = '/certs/client'
        DOCKER_TLS_VERIFY = '1'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }


        stage('Test') {
            steps {
                // Run npm test inside Node container
                sh "docker run --rm -v ${env.WORKSPACE}:/app -w /app ${DOCKER_IMAGE} npm test"
            }
        }

        stage('Build Docker Image') {
            steps {
                // Build Docker image using DinD
                sh "docker build -t my-app:latest ${env.WORKSPACE}"
            }
        }

        stage('Push Docker Image') {
            steps {
                // Push to Docker Hub using credentials
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                    sh 'docker push my-app:latest'
                }
            }
        }

        stage('Snyk Security Scan') {
            steps {
                // Scan using Snyk
                withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {
                    sh "docker run --rm -v ${env.WORKSPACE}:/app -w /app snyk/snyk-cli:docker test --all-projects"
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
