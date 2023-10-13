# Learning CIRCLE CI 

### 1. Build and Push Docker Images Config - Basic
```yaml
version: 2.1

jobs:
  build_and_push:
    docker:
      - image: cimg/python:3.12.0
    steps:
      - checkout
      - setup_remote_docker:
          version: 20.10.14
          docker_layer_caching: true
      - run: |
          TAG=0.1.$CIRCLE_BUILD_NUM
          IMAGE_NAME=$DOCKER_USER/django-app-circleci:$TAG
          docker build -t  $IMAGE_NAME .
          echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
          docker push $IMAGE_NAME

workflows:
  build_push_and_deploy:
    jobs:
      - build_and_push
```

### 2. Detects change in a particular directory (stage) and builds that dir only 

```yaml
version: 2.1

jobs:
  build_and_push:
    docker:
      - image: cimg/python:3.12.0
    steps:
      - checkout
      - setup_remote_docker:
          version: 20.10.14
          docker_layer_caching: true
      - run:
          name: Build and Push Docker Image
          command: |
            if [[ -n $(git diff --name-only $CIRCLE_SHA1 $CIRCLE_SHA1^ | grep "dev/") ]]; then
              # Changes detected in dev directory
              TAG=dev-0.1.$CIRCLE_BUILD_NUM
              IMAGE_NAME=$DOCKER_USER/django-app-circleci-dev:$TAG
              docker build -t $IMAGE_NAME dev
            elif [[ -n $(git diff --name-only $CIRCLE_SHA1 $CIRCLE_SHA1^ | grep "prod/") ]]; then
              # Changes detected in prod directory
              TAG=prod-0.1.$CIRCLE_BUILD_NUM
              IMAGE_NAME=$DOCKER_USER/django-app-circleci-prod:$TAG
              docker build -t $IMAGE_NAME prod
            else
              echo "No relevant changes detected in dev or prod, skipping the build."
              exit 0
            fi

            echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
            docker push $IMAGE_NAME

workflows:
  build_push_and_deploy:
    jobs:
      - build_and_push:
          filters:
            branches:
              only:
                - main  
```