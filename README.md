# Deploy Microservices Applications 

**The flow of **

**Infomation need for Deploy Micro Apps**
```
    - What Micro Service I am deploy ?
    
    - How Micro Service Connect to each other ?
    
    - Micro Service need any 3rd party service or Database ?
    
    - Which services is accessible outside the Cluster ?

    - Which Port Apps are listening ? 
```

**How Application communicate to Each other ?**
```
    - Direct using API call ?

    - Or Message Broker ?

    - Or Services Mesh

    - In case of Message Broker and Services Mesh . I want to know application exactly they are using ? 
```

**Visualize the whole Picture**

<img width="600" alt="Screenshot 2025-02-17 at 14 36 17" src="https://github.com/user-attachments/assets/672c5bd8-d481-4bde-aa8b-65fe41943d8d" />

```
    - In the picture I can see which service connect to which service

    - For example:

        - Recommendation Service need to talk with ProductCatalog Service

        - So Recommdation Service needs the ProductCatalog Services Endpoint = service name + service port and provide it in the Container ENV

        - The Reason Recommand Service can connect to ProductCalatoc Service by Endpoint is K8 built-in networking feature Internal DNS that allow Service can communicate with each other using human-readable-name . This make Service discovery within Cluster automate and Dynamic

    - The same thing for other services that connect to each other 
```

**Image of each Container**

```
    - emailservice: gcr.io/google-samples/microservices-demo/emailservice:v0.8.0
    - paymentservice: gcr.io/google-samples/microservices-demo/paymentservice:v0.8.0
    - shippingservice: gcr.io/google-samples/microservices-demo/shippingservice:v0.8.0
    - adservice: gcr.io/google-samples/microservices-demo/adservice:v0.8.0
    - checkoutservice: gcr.io/google-samples/microservices-demo/checkoutservice:v0.8.0
    - currencyservice: gcr.io/google-samples/microservices-demo/currencyservice:v0.8.0
    - frontend: gcr.io/google-samples/microservices-demo/frontend:v0.8.0
    - productcatalogservice: gcr.io/google-samples/microservices-demo/productcatalogservice:v0.8.0
    - cartservice: gcr.io/google-samples/microservices-demo/cartservice:v0.8.0
    - recommendationservice: gcr.io/google-samples/microservices-demo/recommendationservice:v0.8.0
```

**Configure Deployment and Service for each Container**

- Basic setup

```
apiVersion: v1
kind: Deployment
metadata:
  name: xxxx
spec
  selector:
    matchLabels:
      app: xxxx
  template:
    metadata:
      labels:
        app: xxxx
    spec: 
      container:
      - name: xxxx
        image: xxxx
        ports:
        - containerPort: xxxx
---
apiVersion: v1
kind: Service
metadata:
  name: xxxx
spec: 
  type: ClusterIP
  selector:
    app: xxxx
  ports:
  - protocol: TCP
    port: xxxx
    targetPort: xxxx
```

**Create Linode Cluser**

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

**Create Ingress for External Request**

----Ingress Controller----

```
    - Ingress Controller is another the Component in the Cluster to evaluate all the rules that defined in the Cluster . That way it can manage all the redirection . This will be an Entry point of the Cluster

    - Ingress Controller use some Cloud load balancer in the background

    - Linode'own Node Balancer and Worker Node Balancer , that was dynamically created and provisioned as I created the Ingress Controller 

    1. I use Helm Chart to create Nginx Ingress Controller

    2. Add Ingress Controller (Nginx) Repo to Helm . helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

    3. Installing the Helm chart : helm install [RELEASE_NAME] ingress-nginx/ingress-nginx --set controller.publishService.enabled=true
```

----Ingress Configuration----

```
    - Ingress is a API Object that manage external request to Services . It provide the routing rules so Cluster will know which Request go to which Pod

    - Ingress support multiple hostname

    - Ingress can handle TLS (SSL) termination for HTTPS

    - Ingress support Path Routing

    !!! NOTE: Ingress Controller is the one who run Ingress rules . Not Kubernetes
```
















