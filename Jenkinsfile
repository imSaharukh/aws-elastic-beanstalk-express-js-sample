pipeline {
    agent {
        docker {
            image 'saharukh/node-docker:16' // prebuilt image with node + docker
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        SNYK_TOKEN = credentials('snyk-token')
        DOCKERHUB_REPO = 'yourdockerhub/repo'
        DOCKERHUB_CREDS = 'dockerhub-creds'
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
                sh 'snyk test --severity-threshold=high || true'
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'package.json, package-lock.json', allowEmptyArchive: true
            }
        }
    }
}
