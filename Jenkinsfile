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
                script {
                    docker.image(DOCKER_IMAGE).inside {
                        sh 'npm install'
                    }
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    docker.image(DOCKER_IMAGE).inside {
                        sh 'npm test'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.image('docker:24.0.6-cli').inside('--privileged -v /var/run/docker.sock:/var/run/docker.sock') {
                        sh 'docker build -t my-app:latest .'
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.image('docker:24.0.6-cli').inside('--privileged -v /var/run/docker.sock:/var/run/docker.sock') {
                        sh '''
                            echo $DOCKERHUB_REPO_PSW | docker login -u $DOCKERHUB_REPO --password-stdin
                            docker push my-app:latest
                        '''
                    }
                }
            }
        }

        stage('Snyk Security Scan') {
            steps {
                withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
                    sh 'snyk test'
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
