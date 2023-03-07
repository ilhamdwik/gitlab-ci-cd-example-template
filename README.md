# gitlab-ci-cd-example-template

///
variables:
  IMAGE_NAME: ilhaskam/wayshub-frontend-cicd-gitlab
  IMAGE_TAG: latest
  CONTAINER_NAME: frontend-wayshub-cicd-gitlab

stages:
  - remove_container
  - deploy

remove_container:
  stage: remove_container
  before_script:
    - 'which ssh-agent || ( apt-get update -y && apt-get install openssh-client -y )'
    - mkdir -p ~/.ssh
    - echo "$SSH_KEY" | tr -d '\r' > ~/.ssh/id_ed25519
    - chmod 700 ~/.ssh
    - chmod 400 ~/.ssh/id_ed25519
    - eval $(ssh-agent -s)
    #- ssh-add ~/.ssh/id_ed25519
    #- ssh-keyscan >> ~/.ssh/known_hosts
    #- chmod 644 ~/.ssh/known_hosts
    - '[[ -f /.dockerenv ]] && echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config'
  script:
    - chmod 400 $SSH_PRIVATE_KEY
    - ssh -i $SSH_PRIVATE_KEY -o StrictHostKeyChecking=no $SERVER_USER@$SERVER_IP "
      cd /home/ubuntu/wayshub/wayshub-frontend &&
      /usr/bin/docker stop $CONTAINER_NAME &&
      /usr/bin/docker rm $CONTAINER_NAME"


deploy:
  stage: deploy
  before_script:
    - 'which ssh-agent || ( apt-get update -y && apt-get install openssh-client -y )'
    - mkdir -p ~/.ssh
    - echo "$SSH_KEY" | tr -d '\r' > ~/.ssh/id_ed25519
    - chmod 700 ~/.ssh
    - chmod 400 ~/.ssh/id_ed25519
    - eval $(ssh-agent -s)
    #- ssh-add ~/.ssh/id_ed25519
    #- ssh-keyscan >> ~/.ssh/known_hosts
    #- chmod 644 ~/.ssh/known_hosts
    - '[[ -f /.dockerenv ]] && echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config'
  script:
    - chmod 400 $SSH_PRIVATE_KEY
    - ssh -i $SSH_PRIVATE_KEY -o StrictHostKeyChecking=no $SERVER_USER@$SERVER_IP "
      cd /home/ubuntu/wayshub/wayshub-frontend &&
      /usr/bin/docker login -u $REGISTRY_USER -p $REGISTRY_PASSWORD &&
      /usr/bin/docker build -t $IMAGE_NAME:$IMAGE_TAG . &&
      /home/ubuntu/.local/bin/docker-compose up -d"
///
