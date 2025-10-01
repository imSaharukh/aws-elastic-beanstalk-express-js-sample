pipeline {
  agent {
    docker {
      image 'node:16'
      args '-u root:root'
    }
  }
  environment {
    DOCKERHUB_USER = 'imsaharukh'
  }
  stages {
    stage('Checkout Code') {
      steps {
        checkout scm
      }
    }
    stage('Install Dependencies') {
      steps {
        sh 'npm install --save'
      }
    }
    stage('Run Tests') {
      steps {
        sh 'npm test'
      }
    }
    stage('Build Docker Image') {
      steps {
        sh 'docker build -t $DOCKERHUB_USER/aws-sample-app .'
      }
    }
    stage('Security Scan') {
      steps {
        // Install Snyk CLI inside pipeline
        sh 'npm install -g snyk'
        // Test dependencies
        sh 'snyk test || exit 1'
      }
    }
    stage('Push Image') {
      steps {
        withCredentials([string(credentialsId: 'dockerhub-token', variable: 'DOCKERHUB_PASS')]) {
          sh 'echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin'
          sh 'docker push $DOCKERHUB_USER/aws-sample-app'
        }
      }
    }
  }
}
