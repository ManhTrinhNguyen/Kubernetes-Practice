# Deploy Nodejs App with Kubernetes 

## Clone Nodejs App 

  - `git clone https://gitlab.com/twn-devops-bootcamp/latest/10-kubernetes/js-app.git` 

## Build and Push JS App to Docker hub private Repo 

  - Before I can do those Steps bellow I need Docker Engine set up and running on my Mac
    
  - Step 1 : Build Image : `docker build -t nguyenmanhtrinh/demo-app:js-app-1.0 .`

    -  nguyenmanhtrinh/demo-app : This is my Private repo name
   
    -  js-app-1.0 : Tag name
   
  - Step 2 : Login to DockerHub : `echo "password" |docker login -u nguyenmanhtrinh --password-stdin`

    - Bcs this is Docker Hub I don't need to provide a endpoint to login to my Dockerhub .
   
  - Step 3 : After Login Succeess I want to push Image to Docker Private Repo : docker push nguyenmanhtrinh/demo-app:js-app-1.0

## Practice 1 : Create a Managed Cluster in Linode 

  - Step 1 : Go to Linode sign up an Account

  - Step 2 : After Signed In -> Go to Kubernetes -> Create Cluster
  
  - Step 3 : Choose name -> Choose Region -> Choose 3 Shared CPU
  
  - Step 4 : Connect kubectl to LKE so I can talk to my Cluster in my Local Machine export KUBECONFIG=js-app-cluster-config.yaml
  
  - js-app-cluster-config.yaml : Provided by Linode in the UI
  
  - Now I have Cluster running and I can talk to it via my Local Machine by using kubectl CLI

## Practice 2 : Deploy MongoDB using Helm 

  - Helm is a Kubernetes Package Manager . To package YAML file and distribute them in public and private Repo

  - Helm Chart is a bundle of Yaml .

  - Helm's own Public Repo at ArtifactHub : (https://artifacthub.io/)

  - Step 1 : Install Helm in Mac : `brew install helm`

  - Step 2 : Bitnami is a Provider of Helm Chart . They are also provide and maintain MongoDB Helm Chart in Public . I get Bitnami Repo : `helm repo add bitnami https://charts.bitnami.com/bitnami`

  - To search for Bitnami MongoDB repo : `helm search repo bitnami | grep mongodb`

  !!! Note When I execute Helm chart it will execute aganst my Cluster 

  - Step 3 : Create Value File for MongoDB Helm Chart

    - Bitnami Helm Chart provided default a Value File that I can override . I want to override those value to configure my Logic of my DB
   
    ```
    architecture: replicaset
    replicaCount: 2

    # I use Linode storage Class to automatically create Persistence Volume when it need
    persistence:
      storageClass: "linode-block-storage"
    
    # I define my MongoDB credentials here
    auth: 
      rootPassword: "rootpassword"
      usernames: 
        - "admin"
      passwords: 
        - "password"
    ```

  - Step 4 : After I have my Value file I can start to deploy MongoDB by using Helm

    - To deploy : `helm install <release-name> --values <value-file> bitnami/mongodb`
   
    - To check pods : `kubectl get pods`
   
    - To check all : `kubectl get all`
   
    - To see pod logs if it is running okay : `kubectl logs <pod-name>`
   
     
















