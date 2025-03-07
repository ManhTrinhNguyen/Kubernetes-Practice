# Deploy Java App with Kubernetes


## Practice 1 : Clone Java App from Github

  - `git clone <github-repo-url>`

## Practice 2 : Create Kubernetes Cluster (LKE)

  - Step 1 : Go to Linode sign up an Account

  - Step 2 : After Signed In -> Go to Kubernetes -> Create Cluster

  - Step 3 : Choose name -> Choose Region -> Choose 3 Shared CPU

  - Step 4 : Connect kubectl to LKE so I can talk to my Cluster in my Local Machine `export KUBECONFIG=java-app-cluster-config.yaml`

      - java-app-cluster-config.yaml : Provided by Linode in the UI
   
  - Now I have Cluster running and I can talk to it via my Local Machine by using `kubectl` CLI

## Practice 2 : Deploy MySQL DB with Helm 

  - Helm is a Package Manager for Kubernetes

  - Helm Chart is a bundle of multiple YAML files that written by others . I can take it and use

  - To see any Helm Chart available I go to (https://artifacthub.io)

  - Step 1 : Install Helm for mac `brew install helm`

  - Step 2 : Bitnami is a provider Helm Charts . They also Provide and Maintain MySQL DB Helm chart . To get Bitnami Repo : `helm repo add bitnami https://charts.bitnami.com/bitnami`

  !!! Note : When I execute Helm Command it will execute against the Cluster I connected to 

  - Step 3 : To search for Bitnami Repo : `helm search repo bitnami`

  - Step 4 : Create a Mysql Values files . So I can overwrite the values that I need to for my MySQL

  ```
    architecture: "replication"
    secondary:
      replicaCount: 2
    auth:
      rootPassword: "rootpassword"
      username: "tim"
      password: "password"
    global:
      storageClass: "linode-block-storage"
  ```

  - Step 5 : To install MySQL Helm Charts from Bitnami : `helm install <release-name> --values <mysql-helm-value-yaml-file> bitnami/mysql
`












