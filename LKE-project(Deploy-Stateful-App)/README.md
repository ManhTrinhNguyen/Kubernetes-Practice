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

**Deploy MongExpress - UI for MongoDB**
```
  Step 1 : Create my own Configuration file for 1 Pod and 1 Service

  Step 2 : Once Yaml file created I will create it in the Cluster : kubectl apply -f <yamlfile>

      apiVersion: apps/v1
      kind: Deployment 
      metadata:
        name: mongo-express
        labels:
          app: mongo-express
      spec:
        replicas: 1
        selector: # This is the selector that will be used to match the labels of the pods
          matchLabels:
            app: mongo-express
        template: # This is the pod template
          metadata: # This is the metadata of the pod
            labels:
              app: mongo-express
          spec:  # This is the specification of the pod
            containers: # This is the container that will be running in the pod
            - name: mongo-express # This is the name of the container
              image: mongo-express # This is the image that will be used to create the container
              ports: # This is the port that will be exposed by the container
              - containerPort: 8081
      
              # This is Configuration use to connect to the MongoDB database
              env: # This is the environment variables that will be used by the container
              - name: ME_CONFIG_MONGODB_ADMINUSERNAME # This is the name of MongoDB root username
                value: root
              - name: ME_CONFIG_MONGODB_SERVER # This 
                value: mongodb-0.mongodb-headless # This is a value of MongoDB server
                # This is a endpoint where Mongo Express will connect to the MongoDB Pod . 
                # mongdb-0 is the name of the MongoDB Pod and mongodb-headless is the name of the service that expose the MongoDB Pod
              - name: ME_CONFIG_MONGODB_ADMINPASSWORD # This is the name of MongoDB root password and It is stored in the secret yaml file
                # I use command kubectl get secret mongodb -o yaml to get name of the secret and key of the password
                valueFrom:
                  secretKeyRef:
                    name: mongodb
                    key: mongo-root-password
      ---
      # This is a Internal Service . Only accessible within the cluster
      apiVersion: v1
      kind: Service
      metadata:
        name: mongo-express-service
      spec:
        selector: # This is the selector that will be used to match the labels of the pods
          app: mongo-express
        ports: # This is the port that will be exposed by the service
          - protocal: TCP
            port: 8081 # This is the port that will be exposed by the service
            targetPort: 8081 # This is the port that will be forwarded to the pods


  Step 3 : I use to check that Mongo Express run correctly : kubectl logs <podname>
```

**Deploy Ingress Controller with Helm**

<img width="400" alt="Screenshot 2025-02-25 at 15 08 53" src="https://github.com/user-attachments/assets/ecbe5393-3a1c-4f3c-aeff-608033ca2e95" />

```
  - Ingress Controller is to evaluate all rules that has defined in me Cluster. This way to manage all the redirections. This will be the Entryp point for the Cluster

  - Ingress Cotroller uses some cloud native load balancer in the backgroud

  - Linode's own Node balancer and Worker Node Balancer, that was dynamically created and provisioned as I created the Ingress Controller

  - NodeBalancer is a EntryPoint for my Cluster . It proivde external IP, Port . Request will come first to NodeBlancer then go to Ingress Controller then go to Ingress Rules . Based on the Rules it will forward to which Service for that rule 

  1. Use Helm Chart for Ingress Controller (Nginx)

  2. Add Nginx repo: helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

  3. Installing the Helm Chart : helm install [RELEASE_NAME] ingress-nginx/ingress-nginx --set controller.publishService.enabled=true
      - --set controller.publishService.enabled=true : This attribute to make sure that we are automatically allocated the Public IP for my Ingress Address to use with Nginx

  4. kubectl get pods : To check Ingress Controller running or not 
```

**Deploy Ingress rules**
```
  - Ingress is a API object that manage the external Request . It provide the Routing Rule to manage which pod should request redirect to

  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    annotations: # this is the annotation that will be used by the Ingress Controller
      kubernetes.io/ingress.class: nginx
    name: mongo-express
  spec: 
    rules:
    - host: 23-239-6-26.ip.linodeusercontent.com # Hostname that will be used to access the application
      http: # Define the HTTP forwarding of request coming from host
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: mongo-express-service
                port:
                  number: 8081

```

Browser : hostname configure in Ingress Rule
Hostname get resolved to extetnal IP of NodeBalancer
Ingress Controller resolved the rule and forwared request to internal MongoExpress Service

----Test UI : MongoExpress connected to MongoDB----

I added some data to the MongoDB
Then I delete the pod and restart them : kubectl scale --replicas=0 statefulset/mongodb - But the data still be there bcs I have configured PV - Then I scale back to 3
To see my charts : helm ls
If I done with the chart or reinstall or update I can use : helm uninstall mongodb






