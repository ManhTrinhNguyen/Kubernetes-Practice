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
