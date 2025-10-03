pipeline {
    agent {
        docker {
            image 'node:16'
        }
    }

    environment {
        DOCKERHUB_REPO = credentials('dockerhub-repo-name')
        SNYK_TOKEN = credentials('snyk-token')
    }

    stages {
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
                    docker.withRegistry('', 'dockerhub-creds') {
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
}
