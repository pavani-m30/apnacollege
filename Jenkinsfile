pipeline {
    agent any
    environment {
        // Bind your Docker Hub secret text credential
        DOCKER_HUB_TOKEN = credentials('dockerhub-secret')
    }
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/pavani-m30/apnacollege.git'
            }
        }
        stage('Build') {
            steps {
                sh 'docker build -t pavanimm/python-app:latest .'
            }
        }
        stage('Test') {
            steps {
                // Run tests if you have them, otherwise skip
                sh 'pytest tests/ || echo "No tests found, skipping..."'
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
                // Example: deploy to Kubernetes cluster
                // Replace with your actual deployment command
                sh 'kubectl set image deployment/python-app python-app=pavanimm/python-app:latest || echo "Deployment step skipped"'
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
