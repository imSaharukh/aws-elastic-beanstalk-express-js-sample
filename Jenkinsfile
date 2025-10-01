pipeline {
    agent any
    environment {
        NODE_VERSION = '16'
    }
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/imSaharukh/aws-elastic-beanstalk-express-js-sample'
            }
        }

        stage('Install & Test') {
            steps {
                sh 'docker run --rm -v $PWD:/app -w /app node:$NODE_VERSION npm install --save'
                sh 'docker run --rm -v $PWD:/app -w /app node:$NODE_VERSION npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t my-node-app .'
            }
        }

        stage('Security Scan') {
            steps {
                sh 'npm install -g snyk'
                sh '''
                   snyk test || exit 1
                '''
            }
        }
    }
    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully'
        }
        failure {
            echo 'Pipeline failed. Check logs.'
        }
    }
}
