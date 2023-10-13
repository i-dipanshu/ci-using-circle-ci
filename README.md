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
            function build_and_push_image {
              local directory="$1"
              local tag="$directory-0.1.$CIRCLE_BUILD_NUM"
              local docker_image_name="$DOCKER_USER/django-app-circleci-$directory:$tag"

              if [[ -n $(git diff --name-only $CIRCLE_SHA1 $CIRCLE_SHA1^ | grep "$directory/") ]]; then
                docker build -t "$docker_image_name" "$directory"
    
                echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                docker push "$docker_image_name"
              else
                echo "No relevant changes detected in $directory, skipping the build."
              fi
            }

            # Build and push image for "dev" directory
            build_and_push_image "dev" 

            # Build and push image for "prod" directory
            build_and_push_image "prod"


workflows:
  build_and_push_docker_image:
    jobs:
      - build_and_push:
          filters:
            branches:
              only:
                - main  

```