#!/bin/bash
set -e

# ===================== CONFIG =====================
IMAGE_NAME="pavanimm/python-app"
DOCKERHUB_USER="pavanimm"
DOCKERHUB_REPO="python-app"
KEEP_VERSIONS=3
DOCKERHUB_TOKEN="${DOCKER_HUB_TOKEN}"   # Jenkins credential injected
# ==================================================

echo "====================================="
echo "Collecting existing semantic versions"
echo "====================================="

# Collect semantic versions only (vXX style)
mapfile -t TAGS < <(
  docker images "${IMAGE_NAME}" \
    --format "{{.Tag}}" \
    | grep -E '^v[0-9]+$' \
    | sort -V
)

TOTAL=${#TAGS[@]}

echo "Found versions: ${TAGS[*]}"
echo "Total versions: ${TOTAL}"

# ===================== CLEANUP =====================
if (( TOTAL >= KEEP_VERSIONS )); then
  DELETE_COUNT=$(( TOTAL - KEEP_VERSIONS + 1 ))

  echo "Deleting ${DELETE_COUNT} old version(s)"

  # ---- Get Docker Hub JWT (ONCE) ----
  echo "Authenticating with Docker Hub (JWT)…"
  DOCKERHUB_JWT=$(curl -s -X POST https://hub.docker.com/v2/users/login/ \
    -H "Content-Type: application/json" \
    -d "{\"username\": \"${DOCKERHUB_USER}\", \"password\": \"${DOCKERHUB_TOKEN}\"}" \
    | jq -r .token)

  if [ -z "${DOCKERHUB_JWT}" ] || [ "${DOCKERHUB_JWT}" = "null" ]; then
    echo "❌ Failed to obtain Docker Hub JWT token"
    exit 1
  fi

  for (( i=0; i<DELETE_COUNT; i++ )); do
    OLD_TAG="${TAGS[$i]}"
    echo "---- Deleting version: ${OLD_TAG} ----"

    echo "Stopping containers locally (if any)"
    docker ps -a --filter "ancestor=${IMAGE_NAME}:${OLD_TAG}" -q \
      | xargs -r docker rm -f

    echo "Removing image locally"
    docker rmi -f "${IMAGE_NAME}:${OLD_TAG}" || true

    echo "Deleting tag from Docker Hub"
    curl -f -X DELETE \
      -H "Authorization: JWT ${DOCKERHUB_JWT}" \
      "https://hub.docker.com/v2/repositories/${DOCKERHUB_USER}/${DOCKERHUB_REPO}/tags/${OLD_TAG}/"

    echo "✅ Deleted ${OLD_TAG} from Docker Hub"
  done
else
  echo "No cleanup needed (below retention limit)"
fi

# ===================== VERSIONING =====================
if (( TOTAL == 0 )); then
  NEW_VERSION="v1"
else
  LAST="${TAGS[$((TOTAL - 1))]}"
  NUM="${LAST#v}"
  NEW_VERSION="v$((NUM + 1))"
fi

CONTAINER_NAME="python_app_container_${NEW_VERSION}"

echo "====================================="
echo "Building new version: ${NEW_VERSION}"
echo "====================================="

# ===================== BUILD =====================
docker build -t "${IMAGE_NAME}:${NEW_VERSION}" .
docker tag "${IMAGE_NAME}:${NEW_VERSION}" "${IMAGE_NAME}:latest"

# ===================== PUSH =====================
docker push "${IMAGE_NAME}:${NEW_VERSION}"
docker push "${IMAGE_NAME}:latest"

# ===================== RUN =====================
# Free port 5000 if in use
CONTAINER_ID=$(docker ps -q --filter "publish=5000")
if [ -n "$CONTAINER_ID" ]; then
  echo "Removing container bound to port 5000: $CONTAINER_ID"
  docker rm -f $CONTAINER_ID || true
fi

docker rm -f python-app || true
docker run -d --name "${CONTAINER_NAME}" -p 5000:5000 "${IMAGE_NAME}:${NEW_VERSION}"

echo "====================================="
echo "✅ SUCCESS"
echo "Active versions on local:"
docker images "${IMAGE_NAME}" --format "{{.Tag}}" | sort -V
echo "Container running: ${CONTAINER_NAME}"
echo "====================================="
