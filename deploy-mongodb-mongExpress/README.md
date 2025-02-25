# Kubernetes-Practice

## Deploy MongoDB and Mongo Express

## Demo 
```
  1. Create MongoDB pod 
    - In order to talk to Pod I need Service 
    - I will create Internal Service . Only the components inside cluster can request to it 
  2. Create Mongo Express 
    - Need URL of MongoDB so Mongo Express can connect to it 
    - Credentials : Need username and password to authenticate 
    - The way I can pass these Infomation is via Deployment configuration file through ENV (That is how application configure)
    - I will create a Config Map that containe DB URL and I will create Secret that container Credentials and I will reference both inside the Deployment 
    - Once I have that set up I will need Mongo Express accessiable through browser in order to do that will create external Service that will allow external request to talk to the Pod
```

## Minikube
```
  - Minikube is 1 Node cluster where Control Plane Processes and Worker Prococesses both run in 1 Node

  - To install Minikube : ( https://minikube.sigs.k8s.io/docs/start/ )

  - When Minkube Installed . To start Minikube minikube start --driver docker

  - To interact with cluster I can use kubectl 
```

**1. Create MongoDB Deployment and MongoDB service Yaml**
```
  ----Reason to use Deploy to create Pod----
  - I use Deployment to create Pod bcs It is a blueprint for Replica Mechanism . I want to replica a Pod so when a Pod died and re create and have other replica that is running .
  - No Downtime Happen
  - I can scale Up and Down base on the how many request its need 
  
  ----Services----
  - Each Pod has 1 Service to be as Stable IP address .
  - Bcs when Pod die and re-create new one with new IP address and it cost a lot of time to re-configure IP for connection
  - Also Service can act as a LoadBalancer when I have multiple Replicas. It will handle a request and send the request to the Pod in the same Node and the least busy one .

```

**2. Create Secret**
```
  - From there I will reference value
  - Secrect live in K8s, not it the Repo

  - kubectl apply -f mongo-secret.yaml : Apply secret 
  - kubectl get secret : To check secret

  !!! I have to Apply Secret before apply Deployment

  --- After I applied Secret I will apply mongodb deployment---

  - To apply Deployment and Service : kubectl apply -f <mongodb-yaml file>

  - To check Pod : kubectl get pod

  - To see the Logs of Pod : kubectl logs <pod-name>

  - Also I can use kubectl describe pod <pod-name> to see pod describtion

  - To check Service: kubectl get svc 
```

**3. Create Mongo Express Deployment and Service (External Configuration where will put Data into MongoDB)**


**4. Create Configmap**
```
  - External Configuration
  - Centralized
  - Other component can use it
  - So if I have 2 Application they are using MongoDB then I can just reference that external Configuration. And if i have to change it I will change it in 1 place . Nothing get updated

  kind: Configmap
  metadata/name: a random name
  data: The actual content in key-value pair
```

**Create external Service to access mongo express from browser**
```
  To expose the service externally I need : Loadbalancer Type and NodePort

  - type: LoadBalancer :
    - Assign service an external IP address and so accepts external request
    - LoadBalancer also give a Internal IP

  - nodePort: Port for external IP address must be between 30000-32767

  - ClusterIP: same as internal service type, is default
    - ClusterIP will give the Service an internal IP
    - And LoadBalancer will also give Service an internal IP and also give Service external IP

  In Minikube :
    minikube service <service name> : Assign external service a public IP

    - When command executed Minikube create a tunnel and use localhost http://127.0.0.1 instead of real public IP

    - Minikube behave different actually create a tunnel and uses a localhost address of 127.0.0.1 instead of real IP

    - In the Regualar K8 environment I would access this from the Public IP address associated to the Service 
```
