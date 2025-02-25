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















