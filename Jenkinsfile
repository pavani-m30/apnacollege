pipeline {
    agent any
    environment {
        DOCKER_HUB_TOKEN = credentials('dockerhub-secret')
    }
    stages {
        stage('Clone Repo') {
            steps {
                git 'https://github.com/pavani-m30/apnacollege.git'
            }
        }
        stage('Build Image') {
            steps {
                sh 'docker build -t pavanimm/python-app:latest .'
            }
        }
        stage('Push Image') {
            steps {
                sh '''
                echo $DOCKER_HUB_TOKEN | docker login -u pavanimm --password-stdin
                docker push pavanimm/python-app:latest
                '''
            }
        }
    }
}
