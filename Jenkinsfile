pipeline {
    agent any

    environment {
        DOCKERHUB_REPO = credentials('dockerhub-repo-name')
        SNYK_TOKEN = credentials('snyk-token')
    }

    stages {

        stage('Install Dependencies') {
            steps {
                // Run Node commands inside a Node container using DinD
                sh '''
                    docker run --rm -v $PWD:/app -w /app node:16 npm install --save
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    docker run --rm -v $PWD:/app -w /app node:16 npm test || true
                '''
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
                // Snyk inside Node container
                sh '''
                    docker run --rm -v $PWD:/app -w /app node:16 sh -c "
                        npm install -g snyk &&
                        snyk auth ${SNYK_TOKEN} &&
                        snyk test --severity-threshold=high || true
                    "
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
