# Deploy Nodejs App with Kubernetes 

## Clone Nodejs App 

  - `git clone https://gitlab.com/twn-devops-bootcamp/latest/10-kubernetes/js-app.git` 

## Build and Push JS App to Docker hub private Repo 

  - Before I can do those Steps bellow I need Docker Engine set up and running on my Mac
    
  - Step 1 : Build Image : `docker build -t nguyenmanhtrinh/demo-app:js-app-1.0 .`

    -  nguyenmanhtrinh/demo-app : This is my Private repo name
   
    -  js-app-1.0 : Tag name
   
  - Step 2 : Login to DockerHub : `docker login`

    - Bcs this is Docker Hub I don't need to provide a endpoint to login to my Dockerhub .  
