pipeline {
    agent {
        docker {
            image 'docker:24.0.6-cli' // Docker CLI image
            args '-v /var/run/docker.sock:/var/run/docker.sock -v $WORKSPACE:$WORKSPACE'
        }
    }

    environment {
        SNYK_TOKEN = credentials('snyk-token')
        DOCKERHUB_CREDS = 'dockerhub-creds'
        DOCKERHUB_REPO = 'yourdockerhub/repo'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Node & Dependencies') {
            steps {
                sh '''
                    apk add --no-cache nodejs npm
                    npm install
                '''
            }
        }

        stage('Test') {
            steps {
                sh 'npm test || true'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKERHUB_REPO}:latest ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withDockerRegistry(credentialsId: "${DOCKERHUB_CREDS}", url: '') {
                    sh "docker push ${DOCKERHUB_REPO}:latest"
                }
            }
        }

        stage('Snyk Scan') {
            steps {
                sh '''
                    npm install -g snyk
                    snyk auth $SNYK_TOKEN
                    snyk test --severity-threshold=high || true
                '''
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'package.json, package-lock.json', allowEmptyArchive: true
            }
        }
    }
}
