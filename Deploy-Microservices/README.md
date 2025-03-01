# Deploy Microservices Applications 

**The Project Flow**

```
    - External Request will go to Ingress Controller -> Ingress Controller evaluate for the rule defined in Ingress -> Ingress Controller will redirect the Request to a Target (Service) of the Frontend -> Service will send that request to the Pod 
```

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

## Best Practice for Project

**Best Practice 1: Pinned tag version for each Container Image**

**Best Practice 2: Liveness Probe for each container**

<img width="600" alt="Screenshot 2025-02-28 at 12 38 44" src="https://github.com/user-attachments/assets/394985be-0327-4bdf-aa9a-096e019ef3db" />

```
    - K8 managing its resources but not Container . How to let K8s know wheather the Applications inside the Pod is running ?

    ----Perform Helth check with Liveness Probe----

    - With Liveness Probe K8s will restart the Pod if the Applications crash or has issue

    - In this Applications the developer has added small program can be check wheather the Application healthy or accessible . So I can use that in the Liveness Probe

    - All the Microservices in this Apps using gRPC protocol that why I configured gRPC protocol for it

    - But for Redis I use tpcSocket . The alternative of gRPC protocol
        - With tpcSocket the Kubelet will attempt to open socket to my container on that container Port . If it succeeds to establish a connection on this Port the container is considered to be healthy

    - Third Options is check HTTPS endpoint : If Application has endpoint inside App that expose the health status of Application itself . I could hit that endpoint to check wheather application is healthy or not. This Configuration will tell Kubelet there is an HTTP endpoint on the Application on this Port and this URL I can check wheather the URL healthy or not 
```

**Best Practice 3: ReadinessProbe for each Container**

<img width="600" alt="Screenshot 2025-02-28 at 12 38 44" src="https://github.com/user-attachments/assets/c2e809fc-7d98-4e5f-a411-cb35f3098d83" />

```
    - LivenessProbe help K8s see that Application running successfuly only After the Application started

    - ReadinessProbe help K8s see that Appliocation running successfully during the Application startup

    - ReadinessProbe Let's K8s know if Application is ready to recevice traffic

     - But for Redis I use tpcSocket . The alternative of gRPC protocol
            - With tpcSocket the Kubelet will attempt to open socket to my container on that container Port . If it succeeds to establish a connection on this Port the container is considered to be healthy
    
        - Third Options is check HTTPS endpoint : If Application has endpoint inside App that expose the health status of Application itself . I could hit that endpoint to check wheather application is healthy or not. This Configuration will tell Kubelet there is an HTTP endpoint on the Application on this Port and this URL I can check wheather the URL healthy or not 
```    

**Best Practice 4: Resources Request for Each Container**

```
    - Make sure my Application has enough resources to run
```

**Best Practice 5: Resources Limit Container**

```
    - If not limit Container could take all resources of the Pod . Bcs sometime it has bugs or ininity loop
```

**Best Practice 6: Don't expose NodePort**

```
    - I use Ingress and Load Balancer

    - Request will first go to Cloud Provider Load balancer -> Then go to the Ingress Controller -> Ingress Controller will evaluate all the rules in the Cluster -> If the request is good it will redirect to Service -> Then Service will send the request to Pod by using KubeProxy
```

**Best Practice 7: More than 1 Replica for Deployment**

```
    - If I pod died , there will be another Pod available | No Downtime
```

**Best Practice 8: More Than 1 Worker Node in the Cluster**

```
    - If Server crashed or reboot , My Application will not be accessible

    - Ideally I want each Replica to run on different Node . So I can have backup 
```

**Best Practice 9: Using Label**

**Best Practice 10: Using Namespace to isolate resources**

**Best Practice 11 (Security): Ensure Image are free vulnerability**

```
    - This can happen when I using Library or tools that has soe Vulnerability for my Application inside the Container

    - Manually Scan vulnerability on Image

    - Or Automated scan on build Pipeline
```

**Best Practice 12 (Security): No root User for Container**

```
    - Make sure that I don't have container running in my Cluster have root access capablity

    - This exposed security risk bcs a container with root access can access more resources and do much more on host where it is running

    - Most official Image Do not use root user however some Non official Image may use root user
```

**Best Practice 13 (Security): Update K8 to latest Version**

```
  - Important Security Fixed

  - Bug fixed

  ----Update K8 Version Node by Node----
  - To avoild Application downtime 
```


