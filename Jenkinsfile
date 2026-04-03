pipeline {
    agent any

    environment {
        DOCKER_HUB_TOKEN = credentials('dockerhub-secret')
        DOCKER_USER = "pavanimm"
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
                echo "Building and pushing images v1 to v4..."
                sh '''
                echo $DOCKER_HUB_TOKEN | docker login -u $DOCKER_USER --password-stdin

                for i in 1 2 3 4; do
                  VERSION="v$i"

                  echo "Building image: $VERSION"
                  docker build -t $IMAGE_NAME:$VERSION .

                  echo "Tagging as latest"
                  docker tag $IMAGE_NAME:$VERSION $IMAGE_NAME:latest

                  echo "Pushing $VERSION and latest"
                  docker push $IMAGE_NAME:$VERSION
                  docker push $IMAGE_NAME:latest
                done
                '''
            }
        }

        stage('Cleanup Old Images Locally') {
            steps {
                echo "Keeping only latest 3 images locally..."
                sh '''
                TAGS=$(docker images $IMAGE_NAME \
                  --format "{{.Tag}} {{.CreatedAt}}" \
                  | grep -v latest \
                  | sort -r -k2 \
                  | awk '{print $1}')

                COUNT=0
                for TAG in $TAGS; do
                  COUNT=$((COUNT+1))
                  if [ $COUNT -gt 3 ]; then
                    echo "Removing local image: $TAG"
                    docker rmi -f $IMAGE_NAME:$TAG || true
                  fi
                done

                docker image prune -f
                '''
            }
        }

        stage('Cleanup Old Tags on Docker Hub') {
            steps {
                echo "Keeping only latest 3 tags on Docker Hub..."
                sh '''
                TAGS=$(curl -s -u "$DOCKER_USER:$DOCKER_HUB_TOKEN" \
                  "https://hub.docker.com/v2/repositories/$DOCKER_USER/python-app/tags/?page_size=100" \
                  | jq -r '.results
                    | map(select(.name != "latest"))
                    | sort_by(.last_updated)
                    | reverse
                    | .[].name')

                COUNT=0
                for TAG in $TAGS; do
                  COUNT=$((COUNT+1))
                  if [ $COUNT -gt 3 ]; then
                    echo "Deleting remote tag: $TAG"
                    curl -s -u "$DOCKER_USER:$DOCKER_HUB_TOKEN" \
                      -X DELETE \
                      "https://hub.docker.com/v2/repositories/$DOCKER_USER/python-app/tags/$TAG/"
                  fi
                done
                '''
            }
        }

        stage('Deploy Container') {
            steps {
                echo "Deploying latest container..."
                sh '''
                # Remove container using port 5000
                CONTAINER_ID=$(docker ps -q --filter "publish=5000")
                if [ -n "$CONTAINER_ID" ]; then
                  docker rm -f $CONTAINER_ID || true
                fi

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
