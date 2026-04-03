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

        stage('Build & Push Images') {
            steps {
                echo "Building and pushing 4 images with version tags..."
                sh '''
                echo $DOCKER_HUB_TOKEN | docker login -u pavanimm --password-stdin

                for i in 11 12 13 14; do
                  VERSION="v$i"
                  echo "Building image with tag: $VERSION"
                  docker build -t $IMAGE_NAME:$VERSION .

                  echo "Tagging image also as latest"
                  docker tag $IMAGE_NAME:$VERSION $IMAGE_NAME:latest

                  echo "Pushing image $VERSION and latest"
                  docker push $IMAGE_NAME:$VERSION
                  docker push $IMAGE_NAME:latest
                done
                '''
            }
        }

        stage('Cleanup Old Images') {
            steps {
                echo "Keeping only the latest 3 images locally, removing older ones..."
                sh '''
                TAGS=$(docker images --format "{{.Repository}}:{{.Tag}} {{.CreatedAt}}" | grep $IMAGE_NAME | sort -r -k2 | awk '{print $1}')

                COUNT=0
                for TAG in $TAGS; do
                  COUNT=$((COUNT+1))
                  if [ $COUNT -gt 3 ]; then
                    echo "Removing old image: $TAG"
                    docker rmi -f $TAG || true
                  fi
                done

                # Also prune dangling <none> images
                docker image prune -f
                '''
            }
        }

        stage('Deploy Container') {
            steps {
                echo "Deploying container with latest image..."
                sh '''
                # Check if port 5000 is already in use
                if lsof -i :5000 -sTCP:LISTEN -t >/dev/null; then
                  echo "Port 5000 is in use. Removing container using it..."
                  CONTAINER_ID=$(docker ps -q --filter "publish=5000")
                  if [ -n "$CONTAINER_ID" ]; then
                    docker rm -f $CONTAINER_ID || true
                  fi
                fi

                # Remove old container named python-app if it exists
                docker rm -f python-app || true

                # Run new container on port 5000
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
