stage('Build & Push Images') {
    steps {
        sh '''
        echo $DOCKER_HUB_TOKEN | docker login -u $DOCKER_USER --password-stdin

        for i in 1 2 3 4; do
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

stage('Cleanup Old Images Locally') {
    steps {
        sh '''
        echo "Cleaning up old images, keeping only latest 3..."

        # Get only valid tags (exclude <none>)
        TAGS=$(docker images --format "{{.Repository}}:{{.Tag}} {{.CreatedAt}}" \
          | grep $IMAGE_NAME \
          | grep -v "<none>" \
          | sort -r -k2 \
          | awk '{print $1}')

        COUNT=0
        for TAG in $TAGS; do
          COUNT=$((COUNT+1))
          if [ $COUNT -gt 3 ]; then
            echo "Removing old image: $TAG"
            docker rmi -f $TAG || true
          fi
        done

        # Remove dangling images safely
        docker image prune -f
        '''
    }
}

stage('Cleanup Old Tags on Docker Hub') {
    steps {
        sh '''
        echo "Deleting old tags from Docker Hub, keeping only latest 3..."

        TAGS=$(curl -s -u "$DOCKER_USER:$DOCKER_HUB_TOKEN" \
          "https://hub.docker.com/v2/repositories/$DOCKER_USER/python-app/tags/?page_size=100" \
          | jq -r '.results|sort_by(.last_updated)|reverse|.[].name')

        COUNT=0
        for TAG in $TAGS; do
          COUNT=$((COUNT+1))
          if [ $COUNT -gt 3 ]; then
            echo "Deleting old tag from Docker Hub: $TAG"
            curl -s -u "$DOCKER_USER:$DOCKER_HUB_TOKEN" \
              -X DELETE "https://hub.docker.com/v2/repositories/$DOCKER_USER/python-app/tags/$TAG/"
          fi
        done
        '''
    }
}
