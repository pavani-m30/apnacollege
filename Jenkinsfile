pipeline {
    agent any
    environment {
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
                echo "Building Docker image with version tags..."
                sh '''
                IMAGE_NAME="pavanimm/python-app"
                VERSION="v$(date +%Y%m%d%H%M%S)"

                echo "Building image with tag: $VERSION"
                docker build -t $IMAGE_NAME:$VERSION .

                echo "Also tagging image as latest"
                docker tag $IMAGE_NAME:$VERSION $IMAGE_NAME:latest

                echo "Saving version tag for later stages"
                echo $VERSION > version.txt
                '''
            }
        }

        stage('Push Image') {
            steps {
                echo "Pushing images to Docker Hub..."
                sh '''
                IMAGE_NAME="pavanimm/python-app"
                VERSION=$(cat version.txt)

                echo $DOCKER_HUB_TOKEN | docker login -u pavanimm --password-stdin
                docker push $IMAGE_NAME:$VERSION
                docker push $IMAGE_NAME:latest
                '''
            }
        }

        stage('Deploy Container') {
            steps {
                echo "Deploying container with latest image..."
                sh '''
                IMAGE_NAME="pavanimm/python-app"

                # Remove old container if exists
                docker rm -f python-app || true

                # Run new container with latest tag
                docker run -d --name python-app -p 5000:5000 $IMAGE_NAME:latest
                '''
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
