pipeline {
    agent any
    environment {
        DOCKER_HUB_TOKEN = credentials('dockerhub-secret')
        IMAGE_NAME = "pavanimm/python-app"
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
                VERSION="v$(date +%Y%m%d%H%M%S)"
                docker build -t $IMAGE_NAME:$VERSION .
                docker tag $IMAGE_NAME:$VERSION $IMAGE_NAME:latest
                echo $VERSION > version.txt
                '''
            }
        }

        stage('Push Image') {
            steps {
                echo "Pushing images to Docker Hub..."
                sh '''
                VERSION=$(cat version.txt)
                echo $DOCKER_HUB_TOKEN | docker login -u pavanimm --password-stdin
                docker push $IMAGE_NAME:$VERSION
                docker push $IMAGE_NAME:latest
                '''
            }
        }

        stage('Cleanup Old Images') {
            steps {
                echo "Keeping only the latest 3 images, removing older ones..."
                sh '''
                echo $DOCKER_HUB_TOKEN | docker login -u pavanimm --password-stdin
                # Get all tags sorted by creation date (newest first)
                TAGS=$(docker images --format "{{.Repository}}:{{.Tag}} {{.CreatedAt}}" | grep $IMAGE_NAME | sort -r -k2 | awk '{print $1}')
                
                COUNT=0
                for TAG in $TAGS; do
                  COUNT=$((COUNT+1))
                  if [ $COUNT -gt 3 ]; then
                    echo "Removing old image: $TAG"
                    docker rmi -f $TAG || true
                  fi
                done
                '''
            }
        }

        stage('Deploy Container') {
            steps {
                echo "Deploying container with latest image..."
                sh '''
                docker rm -f python-app || true
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
