# Deploying Docker Image from Private Repo

## Minikube

Step 1 : Install Minikube : ( https://minikube.sigs.k8s.io/docs/start/ )

  -- Minikube is 1 Node Cluster where Control Plan and Work Node run in 1 Node 

Step 2 : Create and Start Minikube : minikube start --driver docker 

  -- To Interact with Minikube I use kubectl
  -- kubectl is a CLI tool for K8 

## Deploy Images In K8s from Docker Private Repo

**Common Flow**

  -- Commit code to Git -> Trigger CI build (Jenkins) -> Package App with its Environment into Docker Image -> Then Docker Images get push to Docker Repo (AWS, Nexus, Dockerhub ...)

  -- Now I have Docker Image in my Private Repo . I want to use it in my K8s 

**Step to pull Images from Private Repo**

Step 1 : Create Secret Component: Contain access token or crenditals to my Docker Private Repo

Step 2 : Configure Deployment/Pod : To use that Secret using Attribute called imagePullSecrets

**Set up**

  -- Docker Image in ECR (Nodejs Application)

  -- Inside the Repo I have 2 Images with 2 different tag

  -- Locally I have Minikube cluster setup (it's empty)

**Step 1 : Create Secret Component**

  -- This Secret Component need to have credentials to Private Repository (ECR) which will allow Docker to pull that Image

  ----First thing I need to login to Docker Repo----

  -- Create config.json for Secret 
  
  -- Login into Docker Private Repo : aws ecr get-login-password --region eu-central-1 | docker login --username AWS --password-stdin .....

  -- After Login succeeded -> In the background it automatically created config.json file (in .docker/config.json) that hold the authentication to my Private Repo 

  ----There is 2 ways that this config.json will store Authentication----

    -- Is in .docker/config.json
    -- Or in externally "credsStore":"desktop" -> More secure bcs access token will store in Credential Store 

  -- Now whenever Docker tries to pull the Image from Private Repo . It will use those credentials in config.json for this Private Registry to pull that Image to authenticate itself and pull that Image 

  -- However I'm using Minikube and Minikube doesn't have access to CRED Store bcs it is running in Virtual Box. When the Docker which is packaged in the Minikube . So 
Minikube has its own Docker so when Docker inside Minikube tries to pull that Image from this private Repo . It won't see CRED store and it won't be able to access that . 

  ----Different method to authenticate to my Private Repo via Minikube----

    -- I execute command `aws ecr get-login-password` to get Login password (A token use to login AWS)
    -- Then I ssh to Minikube `minikube ssh`
    -- In /home/docker I execute `docker login --username AWS -p <AWS token> <URL to my AWS Repo>`
    -- I will use .docker/config.json inside minikube to create Secrect Component

  ----Create Secret Component----
  
    -- Create Secret component file
    -- Copy config.json file inside minikube to my Local Machine : `minikube cp minikube:/home/docker/.docker/config.json /users/trinhnguyen/.docker/config.json`
    -- Turn config.json file into base 64 : `cat .docker/config.json | base64`
    -- I also can configure Secret using kubectl: `kubectl create secret generic my-generic-key --from-file-.dockerconfigjson=.docker/config.json --type=kubernetes.io/dockerconfigjson`

**Step 2 : Create Deployment**
  -- Pull Docker Image from Private Repo : `imagePullPolicy: always`
















  
