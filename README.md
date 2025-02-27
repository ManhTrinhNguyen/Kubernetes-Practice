# Deploy Microservices Applications 

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
















