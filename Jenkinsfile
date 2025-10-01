pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/imSaharukh/aws-elastic-beanstalk-express-js-sample', branch: 'main'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'docker run --rm -v $PWD:/app -w /app node:16 npm install'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'docker run --rm -v $PWD:/app -w /app node:16 npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t my-app-image .'
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
