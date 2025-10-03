pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'node:16'
        DOCKERHUB_REPO = credentials('dockerhub-repo-name')
        SNYK_TOKEN = credentials('snyk-token')
    }
    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Install Dependencies') {
            steps {
                sh "docker run --rm -v ${env.WORKSPACE}:/app -w /app ${DOCKER_IMAGE} npm install"
            }
        }

        stage('Test') {
            steps {
                sh "docker run --rm -v ${env.WORKSPACE}:/app -w /app ${DOCKER_IMAGE} npm test"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t my-app:latest ${env.WORKSPACE}"
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                    sh 'docker push my-app:latest'
                }
            }
        }
    }
}
