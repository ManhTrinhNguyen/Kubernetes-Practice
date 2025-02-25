# Deploy MongoDB and MongExpress with Helm 

**Overview**
```
  - I will deploy replicated DB (MongoDB). And make it accessible from a UI client (Mongo Express) Browser using Ingress

  1. Deploy MongoDB in Linode Using Helm

  2. Created replicated MongoDB using Statefulset . Also configure PV for DB using Linode Cloud Storage

  3. Deploy UI Mongo Express in order to access from Browser

  4. For this client I will configure Nginx Ingress -> Deploy Ingress controller in the cluster and configure Ingress in order to demonstrate handling browser request in the Cluster 
```

**Helm**
```
  - Helm is a package Manager for Kubernetes to package YAML file and distribute them in public and private Repo 

  ----What is Helm Chart ?----
  - Helm Chart is a bundle of YAML file
  - Using Helm to Create Helm Chart and push to Helm Repo
  - Or Download and Use a existing one

  ----Templating Engine----
  - If I have multiple Microservices are pretty much the same and the different is Application Name, version or docker images and version tag so I can use Template 
  - Template Engine is to define common blueprint for all Microservices
  - Dynamic Values replace by placeholdeer and that would be Template file

    ----Value Injection----
    - I can overwrite the default Value

  ----Release Managment----
  - Manage by Helm binary
  - helm upgrade <chartname> executed : Any change since installation of the last Release are going to be applied . Simple remove it and create new one
  - helm rollback <chartname> executed : I can rollback that upgrade . I can also rollback to the specific release version of the chart 
```

**Create K8 Cluster on Linode**
```
  Step 1 : In the UI choose Kubernetes 

  Step 2 : 
    - Choose Lable
    - Region
    - Kubernetes Version 
    - Choose Node: Worker Node (Linode has already taken care of Control Plane . I don't need to set up Control Plane)
    - Choose Shared CPU
    - Choose Capacity 

  Step 3 : While Nodes are running 
    - I need access credentials to access from my local machine so I download Kubeconfig credentials that Linode provided
    - Change Permission to only my User can read from the file : `chmod 400 kubeconfig`
    - Set kubeconfig.yaml as ENV `export KUBECONFIG=kubeconfig.yaml` -> Then my local machine connect to Nodes 
```

**Deploy MongoDB with Statefulset with Helm**
```
  Step 1 : Install Helm on Mac : brew install helm

  Step 2 : Bitnami maintain MongoDB Helm Chart . I will install the Bitnami Repo : helm repo add bitnami https://charts.bitnami.com/bitnami

  Step 3 : To search Bitnami Repo : helm search repo bitnami -- Search for MongoDB helm chart in bitnami : helm search repo bitnami/mongodb

  !!! !!! Note : When I execute helm command It is going to execute it against the Cluster that I am connected to

  Step 4 : Installing the Chart
    - When I am installing the chart there might be some value that I want to overwrite -> Chart provide some default value and I want to check what Parameters or Value I can overwrite:
    - Example Yaml file that I want to overwrite some value : 
      1. Define to run Statefulset with : architecture
      2. Set root password : auth.rootPassword
      3. Configure Volume -> Helm Chart should use the StorageClass of Linode Cloud Storage
      4. Install the MongoDB chart : helm install mongodb --values helm-mongodb.yaml bitnami/mongodb

    - After I execute helm install mongodb --values helm-mongodb.yaml bitnami/mongodb . It will install mongodb chart in my CLuster

  Step 5 : Check My Cluster
    - Check pod : kubectl get pod
    - Check all : kubectl get all
    - Check Secret : kubectl get secret
      -> This Secret contain root password I provide in Yaml File

  ----What Happen in the Background----
  - What happen is when Statefulset was created for each pod, a physical storage was created for each of three pods
  - And for each Physical Storage, a persistence volume was created and that is now attached to the Node where Pod is running 

```












