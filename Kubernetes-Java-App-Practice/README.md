# Deploy Java App with Kubernetes


## Practice 1 : Clone Java App from Github

  - `git clone <github-repo-url>`

## Practice 2 : Create Kubernetes Cluster (LKE)

  - Step 1 : Go to Linode sign up an Account

  - Step 2 : After Signed In -> Go to Kubernetes -> Create Cluster

  - Step 3 : Choose name -> Choose Region -> Choose 3 Shared CPU

  - Step 4 : Connect kubectl to LKE so I can talk to my Cluster in my Local Machine `KUBECONFIG=java-app-cluster-config.yaml`

      - java-app-cluster-config.yaml : Provided by Linode in the UI  
