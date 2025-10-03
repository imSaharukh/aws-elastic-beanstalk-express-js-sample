pipeline {
    agent {
        docker {
            image 'node:16'
            args '-u root:root' // run as root to avoid permission issues
        }
    }

    environment {
        DOCKERHUB_REPO = credentials('dockerhub-repo-name')  // Docker Hub repo (username/repo)
        DOCKERHUB_CREDS = 'dockerhub-creds'                  // Docker Hub credentials ID
        SNYK_TOKEN = credentials('snyk-token')              // Snyk token
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install --save'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test || true'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKERHUB_REPO}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('', "${DOCKERHUB_CREDS}") {
                        docker.image("${DOCKERHUB_REPO}").push("latest")
                    }
                }
            }
        }

        stage('Snyk Security Scan') {
            steps {
                sh '''
                npm install -g snyk
                snyk auth ${SNYK_TOKEN}
                snyk test --severity-threshold=high || true
                '''
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'package.json, package-lock.json, **/npm-debug.log, .snyk, snyk-result.json', allowEmptyArchive: true
            }
        }
    }

    post {
        always {
            cleanWs()  // clean workspace after build
        }
    }
}
