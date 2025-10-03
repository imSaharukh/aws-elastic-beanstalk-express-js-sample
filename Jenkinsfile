pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'imsaharukh/project2-compose:latest'
        SNYK_TOKEN = credentials('snyk-token')
        DOCKER_HOST = 'tcp://docker:2376'
        DOCKER_CERT_PATH = '/certs/client'
        DOCKER_TLS_VERIFY = '1'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image ${env.DOCKER_IMAGE}"
                sh "docker build -t ${DOCKER_IMAGE} ${env.WORKSPACE}"
            }
        }

        stage('Push Docker Image') {
            steps {
                echo "Pushing Docker image ${env.DOCKER_IMAGE} to Docker Hub"
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                    sh "docker push ${DOCKER_IMAGE}"
                }
            }
        }

        stage('Snyk Security Scan') {
            steps {
                echo 'Running Snyk security scan'
                withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {
                    sh '''
                    npm install -g snyk
                    snyk auth $SNYK_TOKEN
                    snyk test --all-projects
                    '''
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
