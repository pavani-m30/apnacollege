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
                sh 'docker build -t pavani30/python-app:latest .'
            }
        }
        stage('Push Image') {
            steps {
                sh '''
                echo $DOCKER_HUB_TOKEN | docker login -u pavani30 --password-stdin
                docker push pavani30/python-app:latest
                '''
            }
        }
    }
}

