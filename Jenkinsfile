pipeline {
    agent {
        kubernetes {
      label 'multi-branch-nodejs'
      defaultContainer 'node'
      yaml """
apiVersion: v1
kind: Pod
metadata:
labels:
  component: multi-branch-nodejs
spec:
  # Use service account that can deploy to all namespaces
  serviceAccountName: cd-jenkins
  containers:
  - name: node
    image: node:9-alpine
    command:
    - cat
    tty: true
"""
}
    }
    environment {
        CI = 'true'
    }
    stages {
        stage('Build') {
            steps {
                sh 'chown -R $USER:$(id -gn $USER) /.config'
                sh 'npm config set unsafe-perm=true'
                sh 'npm install'
            }
        }
        stage('Test') {
            steps {
                sh './jenkins/scripts/test.sh'
            }
        }
        stage('Deliver for development') {
            when {
                branch 'development'
            }
            steps {
                sh './jenkins/scripts/deliver-for-development.sh'
                input message: 'Finished using the web site? (Click "Proceed" to continue)'
                sh './jenkins/scripts/kill.sh'
            }
        }
        stage('Deploy for staging') {
            when {
                branch 'staging'
            }
            steps {
                sh './jenkins/scripts/deploy-for-staging.sh'
                input message: 'Finished using the web site? (Click "Proceed" to continue)'
                sh './jenkins/scripts/kill.sh'
            }
        }
        stage('Deploy for production') {
            when {
                branch 'production'
            }
            steps {
                sh './jenkins/scripts/deploy-for-production.sh'
                input message: 'Finished using the web site? (Click "Proceed" to continue)'
                sh './jenkins/scripts/kill.sh'
            }
        }
    }
}
