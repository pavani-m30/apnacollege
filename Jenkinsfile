pipeline {
    agent any
    environment {
        DOCKER_HUB_TOKEN = credentials('dockerhub-secret')
    }
    stages {
        stage('Checkout') {
            steps {
                // Pull code from GitHub
                git branch: 'main', url: 'https://github.com/pavani-m30/apnacollege.git'
            }
        }
        stage('Build') {
            steps {
                sh 'docker build -t pavanimm/python-app:latest .'
            }
        }
        stage('Push') {
            steps {
                sh '''
                echo $DOCKER_HUB_TOKEN | docker login -u pavanimm --password-stdin
                docker push pavanimm/python-app:latest
                '''
            }
        }
        stage('Deploy') {
            steps {
                // Clean up old container if exists
                sh 'docker rm -f python-app || true'

                // Run new container on port 5000
                sh 'docker run -d --name python-app -p 5000:5000 pavanimm/python-app:latest'
            }
        }
    }
    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed. Check logs.'
        }
    }
}
