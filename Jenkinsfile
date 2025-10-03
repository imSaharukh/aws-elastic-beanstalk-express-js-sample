pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'my-app-node:latest' // custom image we will build
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

        stage('Build Node Test Image') {
            steps {
                script {
                    sh '''
                        cat > Dockerfile.test <<EOF
        FROM node:20
        WORKDIR /app
        COPY . .
        RUN npm install
        CMD ["npm", "test"]
        EOF
                    '''
                    sh "docker build -t ${DOCKER_IMAGE} -f Dockerfile.test ."
                }
            }
        }


        stage('Test') {
            steps {
                // Run tests using the custom image
                sh "docker run --rm ${DOCKER_IMAGE}"
            }
        }

        stage('Build Docker Image') {
            steps {
                // Build Docker image for deployment
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
