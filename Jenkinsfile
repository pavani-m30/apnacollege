pipeline {
    agent any
    environment {
        // Docker Hub credentials stored in Jenkins
        DOCKER_HUB_TOKEN = credentials('dockerhub-secret')
    }
    stages {
        stage('Checkout') {
            steps {
                echo "Pulling code from GitHub..."
                git branch: 'main', url: 'https://github.com/pavani-m30/apnacollege.git'
            }
        }

        stage('Build Image') {
            steps {
                echo "Building Docker image..."
                sh 'docker build -t pavanimm/python-app:latest .'
            }
        }

        stage('Push Image') {
            steps {
                echo "Logging in and pushing image to Docker Hub..."
                sh '''
                echo $DOCKER_HUB_TOKEN | docker login -u pavanimm --password-stdin
                docker push pavanimm/python-app:latest
                '''
            }
        }

        stage('Deploy Container') {
            steps {
                echo "Cleaning up old container if exists..."
                sh 'docker rm -f python-app || true'

                echo "Checking if port 5000 is in use..."
                sh '''
                CONTAINER_ID=$(docker ps --filter "publish=5000" --format "{{.ID}}")
                if [ -n "$CONTAINER_ID" ]; then
                  echo "Stopping container $CONTAINER_ID using port 5000..."
                  docker stop "$CONTAINER_ID"
                  docker rm "$CONTAINER_ID"
                fi
                '''

                echo "Running new container..."
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
