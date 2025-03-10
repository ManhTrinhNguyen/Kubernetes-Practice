# Deploy Java App with Kubernetes

## Build and Push Docker Image to ECR 

  - Step 1 : Create Docker file to build Image

  - Step 2 : Create ECR Repository

  - Step 3 : Build Docker image with ECR Repo URL tag : `docker build -t 565393037799.dkr.ecr.us-west-1.amazonaws.com/java-app:1.0`

    - Build Image with ECR Repo URL tag bcs to ensure that image correctly pushed, stored ,deployed in AWS
   
  - Step 4 : Login to AWS : `aws ecr get-login-password --region us-west-1 | docker login --username AWS --password-stdin 565393037799.dkr.ecr.us-west-1.amazonaws.com`
  
    - In able to Login to to my aws I need AWS CLI installed and Credentials Configured  
   
  - Step 5 : Push Image to AWS . `docker push 565393037799.dkr.ecr.us-west-1.amazonaws.com/java-app:1.0`  

## Practice 1 : Clone Java App from Github

  - `git clone <github-repo-url>`

## Practice 2 : Create Kubernetes Cluster (LKE)

  - Step 1 : Go to Linode sign up an Account

  - Step 2 : After Signed In -> Go to Kubernetes -> Create Cluster

  - Step 3 : Choose name -> Choose Region -> Choose 3 Shared CPU

  - Step 4 : Connect kubectl to LKE so I can talk to my Cluster in my Local Machine `export KUBECONFIG=java-app-cluster-config.yaml`

      - java-app-cluster-config.yaml : Provided by Linode in the UI
   
  - Now I have Cluster running and I can talk to it via my Local Machine by using `kubectl` CLI

## Practice 3 : Deploy MySQL DB with Helm 

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
  - Step 6 : After MySQL DB started . To debug if something wrong with the pods : `kubectl logs <pods-name>`

  - Step 7 : I Can also get inside the pod to see mysql pod ENV : `kubectl exec -it <pod-name> -- bin/bash` Or `kubectl describe pod <pod-name>`

## Practice 4 : Deploy Java Apps with 2 Replica 

  - Step 1 : Create Deployment

  ```
    - I have Java Image in my ECR Private Repo

    - To pull Image From ECR are I need to create Secret Component containe access token and crenditals to my Private Docker Repo
   
    - Configure Deployment to use that Secret using Attribute called `imagePullSecrets` 
  ```

  - Step 1.2 : Credentials need to Create Secret Component 
  
    - This Secret component need to have credentials to Private Repo (ECR) which allow Docker to pull Image 
  
    - First I need to login to ECR 'aws ecr get-login-password --region us-west-1 | docker login --username AWS --password-stdin 565393037799.dkr.ecr.us-west-1.amazonaws.com'
  
    - After Login succeed , In the background It will automatically create the .docker/config.json . This file store the credentials to login in AWS  . 
  
    - Now Whenever Docker try to pull Image from ECR . It will use those Credentials in config.json to authenticate itself and pull Image
   
    - But K8's doesn't support 
   
  - Step 1.3 : Create Secret Component

    - Bcs K8's doesn't Support credsStore I need to manually create the valid Config.json
   
      ```
        echo '{
        "auths": {
          "565393037799.dkr.ecr.us-west-1.amazonaws.com": {
            "username": "AWS",
            "password": "'$(aws ecr get-login-password --region us-west-1)'",
            "auth": "'$(echo -n "AWS:$(aws ecr get-login-password --region us-west-1)" | base64)'"
          }
        }
      }' > .docker/config.json
      ```
 
  ```
    apiVersion: v1
    kind: Secret
    metadata:
      name: docker-authenticate-secret
    type: kubernetes.io/dockerconfigjson
    data:
      .dockerconfigjson: <My config.json base64 encode>
  ```

  - Step 1.4 : The second way to create Secrect Component

      ```
        kubectl create secret docker-registry <my-secrect-name> \
        --docker-server=https://565393037799.dkr.ecr.us-west-1.amazonaws.com
        --docker-username=AWS
        --docker-password=aws ecr get-login-password --region us-west-1
      ```

  - Step 1.5: ENV for the Container to Connect to DB

    - I Created the Secret Component to store DB_USER, DB_NAME
      
    <img width="500" alt="Screenshot 2025-03-08 at 11 23 58" src="https://github.com/user-attachments/assets/6336f486-5782-4dd1-b0dc-f6f18b508ae8" />

    - And I created Configmap Component to Store DB_URL_Server
   
   <img width="500" alt="Screenshot 2025-03-08 at 11 23 44" src="https://github.com/user-attachments/assets/e2d51da8-5707-4455-8042-6da89198cc37" />

    
    ```
      - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: java-secret
              key: DB_USER
         - name: DB_PWD
          valueFrom:
            secretKeyRef:
              name: mysql
              key: mysql-password
        - name: DB_NAME
          valueFrom:
            secretKeyRef:
              name: java-secret
              key: DB_NAME
        - name: DB_SERVER
          value: mysql-primary-0.mysql-primary-headless
    ```

    - name: DB_SERVER
      value: mysql-primary-0.mysql-primary-headless : This is a value of MySQL server . This is a endpoint where My Java App will connect to MySQL Pod

      - mysql-primary-0 : Pod Name
      - mysql-primary-headless : headless service name 

  - Step 2 : Create Services file

  ```
    apiVersion: v1
    kind: Service
    metadata:
      name: java-app
    spec:
      selector:
        app: java-app
      ports:
        - protocol: TCP
          port: 8080
          targetPort: 8080
  ```
    
## Practice 5 : Deploy phpmyadmin 

-  Step 1: Get phpmyadmin Image from Dockerhub `phpmyadmin:5.2.2-fpm-alpine`

-  Step 2 : Configure Deployment Yaml

    ```
    apiVersion : apps/v1
    kind: Deployment
    metadata:
      name: phpmyadmin
      labels:
        app: phpmyadmin
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: phpmyadmin
      template:
        metadata:
          labels:
            app: phpmyadmin
        spec:
          containers:
          - name: phpmyadmin
            image: phpmyadmin:5.2.2-fpm-alpine
            ports: 
              - containerPort: 8080
            env: 
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql
                  key: mysql-root-password
            - name: PMA_HOST
              value: mysql-primary-0.mysql-primary-headless
            - name: MYSQL_USER
              valueFrom:
                secretKeyRef:
                  name: java-secret
                  key: DB_USER
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql
                  key: mysql-password
      ```
    

    - If I start the Pod but I have error sometime the message like this: 
      
    <img width="500" alt="Screenshot 2025-03-08 at 10 54 05" src="https://github.com/user-attachments/assets/a1825614-146e-4d3c-94ac-2657e2d72f6a" />

    - To see what happen I use : `kubectl logs <pod-name>` .
 
    - Now I know the Error from when It it is trying to start the container . To see what happen inside the pod I can use `kubectl describe pod` .
 
    - Inside the pod describe I see that I miss spelling my mysql secrect in the ENV section :
      
   <img width="500" alt="Screenshot 2025-03-08 at 10 58 34" src="https://github.com/user-attachments/assets/cdc42ced-b6e5-492c-ae45-c3679395829f" />

  - Step 3 : Create Service

  ```
  apiVersion: v1
  kind: Service
  metadata:
    name: phpmyadmin
  spec:
    selector:
      app: phpmyadmin
    ports:
      - protocol: TCP
        port: 80
        targetPort: 8080
  ```

  - Step 4 : To access my phpmyadmin in the UI in my localhost . However I don't want to expose phpmyadmin for Security . I will configure Port-forwarding for the Service to access on localhost : `kubectl port-forward svc/phpmyadmin-service 8081:8081` 

## Practice 6 : Deploy Ingress Controller by using Helm 

```
  - Ingress controller is another Component in the Cluster to evaluate all the Rules that I defined in the Cluster . That way to manage the Redirection

  - Ingress Controller use cloud native in the background . So whenever I installed Ingress Controller my Cloud Loadbalancer automatic created
```

  - Step 1 : Install helm : `brew install helm`

  - Step 2 : Add Nginx Repo : `helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx`

  - Step 3 : Install Ingress controller from Nginx Repo : `helm install <release-name> ingress-nginx/ingress-nginx --set controller.publishService.enabled=true`

    - --set controller.publishService.enabled=true : This attribute to make sure that we are automatically allocated the Public IP for my Ingress Address to use with Nginx
   
    - Step 3 : To check Ingress Controller is running :
      
      <img width="500" alt="Screenshot 2025-03-08 at 13 24 32" src="https://github.com/user-attachments/assets/f6e20416-9969-4ce1-ae9f-43613d7622e6" />
      
    <img width="500" alt="Screenshot 2025-03-08 at 13 25 04" src="https://github.com/user-attachments/assets/92655dae-7bb4-4170-9921-6f78dbdbcfa8" />

      - `kubectl get pods` to see the Ingress controller pod
      
      - `kubectl get svc` to see Ingress Controller Services 

## Practice 7: Define Ingress Rule 

```
  - Ingress is an abstract Component in the Cluster . It provide the Routing rules to manage the redirection . Manage which request redirect to which Pod
```

  - Step 1 : Create Ingress yaml

  ```
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: java-ingress
  spec:
    ingressClassName: nginx
    rules:
    - host: 45-79-231-7.ip.linodeusercontent.com # I use this for host because I don't have a domain name . For test purposes
      http:
        paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: java-app-service
              port:
                number: 8080

  ```


  !!! Error occur : When I try apply Ingress `kubectl apply -f yaml-config-files/java-ingress.yaml`

  ```
  Error from server (InternalError): error when creating "yaml-config-files/java-ingress.yaml": Internal error occurred: failed calling webhook "validate.nginx.ingress.kubernetes.io": failed to call webhook: Post "https://ingress-controller-ingress-nginx-controller-  admission.default.svc:443/networking/v1/ingresses?timeout=10s": context deadline exceeded
  ```

  <img width="600" alt="Screenshot 2025-03-09 at 10 09 38" src="https://github.com/user-attachments/assets/1d3446d0-e38e-4def-a5ee-39588d61beb1" />

  - Then I checked my Ingress Controller and my Cluster . Everything running fine 

  - I go to check the Ingress-Nginx yaml file : `kubectl get pod <ingress-controller-ingress-nginx-controller> -o yaml`

  - Make sure the Ingress-controller Pod is Up, Running, and Ready : `kubectl wait --for=condition=ready pod --selector=app.kubernetes.io/component=controller --timeout=120s` . The Output is "pod/ingress-controller-ingress-nginx-controller-577754b989-fk8mh condition met" . That mean it is ready

    - I am still don't know what happen to the Error . But I destroy the whole Cluster and rent more CPU and deploy it again . Now I can create Ingress .
   

## Practice 8: Create Helm chart for Java App

```
  - Helm Chart is a bundle of Yaml file

  - When I deploy an App with those Configuration Yaml file I can bundle it for the next use 
```

  - Step 1 : To create Helm chart : `helm create <helm-chart-name>`

  - Step 2 : Create Helm chart (Deployment)

  ```
  - Instead of hardcode Values I use syntax for dynamically Values : : {{.Values.<name-of-the-value>}}

  - This is a syntax to configura LivenessProbe and ReadinessProbe or anything need correct Yaml file : ``{{- toYaml.Values.livenessProbe | nindent 12}}``

    -- toYaml : ensure the LivenessProbe or ReadinessProbe is properly formatted instead of being rendered as a single-line string
    -- nindent 12 : Helm Template require correct indention inside deployment.yaml, and 'nindent 12' correctly formatted it at 12 space
  ```

  **Dynamic ENV**
  
  - For 1 single ENV I can have container ENV like this :

  ```
  env:
  - name : {{ .Values.containerEnvVar.key }}
    value : {{ .Values.containerEnvVar.value }}
  ```

  - For working with List of something . In this case ENV, Helm has built function call Range

  - Range is basically loop through a List give me a Element one by one

  - Syntax for Range . This way I can access the key value pair that I define in the Container ENV var List

  ```
  env:
  {{ - range $key, $value := .Values.containerEnvVars}}
  - name: {{ $key }}
    value: {{ $value | quote }}
  {{ - end}}
  
  Note: Value ENV variable alway interpreted as strings . So I use a built-in function called quote and will use piping syntax |
  
  - Piping syntax in Linux : is the way to pass Output of 1 command to other Input of other command
  ```

  **Dynamic ENV in this project**

  - In this project I have 3 Lists of Dynamic ENV : secretData list, configData list, regularData list . So I will configure like this :

  ```
  env:
  # This is for regular Data
  {{- range $key, $value := .Values.regularEnvData }} 
  - name: {{ $key }}
    value: {{ $value | quote }}
  {{ - end }}

  # This is for secretData
  {{- range $key, $value := .Values.secretData }} 
  - name: {{ $key }}
    valueFrom: 
      secretKeyRef:
        # in loop, we lose global context, but can access global context with $
        # $ is 1 variable that is always global and will always point to the root context
        # so $.Value instead of .Values
        name: {{ $.Values.secretName }}
        key: {{ $key }}
  {{- end }}

  # This is for configData
  {{- range $key, $value := .Values.configData }}
  - name: {{ $key }}
    valueFrom:
      configMapKeyRef:
        # in loop, we lose global context, but can access global context with $
        # $ is 1 variable that is always global and will always point to the root context
        # so $.Value instead of .Values
        name: {{ $.Values.configName }}
        key: {{ $key }}
  {{- end }}
  ```

  **Deployment Configuration**
  
  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: {{ .Values.appName }}
    labels:
      app: {{ .Values.appName }}
  spec:
    replicas: {{ .Values.replicasCount }}
    selector:
      matchLabels:
        app: {{ .Values.appName }}
    template:
      metadata:
        labels:
          app: {{ .Values.appName }}
      spec:
        imagePullSecrets:
        - name: {{ .Values.imagePullSecrets }}
        containers:
        - name : {{ .Values.appName }}
          image: "{{ .Values.imageName }}:{{ .Values.imageVersion | toString }}"
          imagePullPolicy: Always
          ports: 
          - containerPort: {{ .Values.containerPort }}
          env: 
          {{- range $key, $value := .Values.regularEnvData }}
          - name: {{ $key }}
            value: {{ $value | quote }}
          {{- end }}
  
          {{- range $key, $value := .Values.secretData }}
          - name: {{ $key }}
            valueFrom: 
              secretKeyRef: 
                name: {{ $.Values.secretName }}
                key: {{ $key }}
          {{- end }}
  
          {{- range $key, $value := .Values.configData }}
          - name: {{ $key }}
            valueFrom:
              configMapKeyRef:
                name: {{ $.Values.configName }}
                key: {{ $key }}
          {{- end }}
  ```  

  - Step 3 : Create Service

  ```
  apiVersion: v1
  kind: Service
  metadata:
    name : {{ .Values.appName }}
  spec:
    selector:
      app: {{ .Values.appName }}
    ports: 
    - protocol: TCP 
      port: {{ .Values.servicePort }}
      target: {{ .Values.containerPort }}
  ```

  - Step 4 : Create ConfigMap

    - I also use Range to Loop the List of Configmap Data . In case I have multiple data
  
  ```
  apiVersion : v1
  kind: ConfigMap 
  metadata:
    name: {{ .Values.configName }}
  data:
    {{- range $key, $value := .Values.configData }}
    {{ $key }}: {{ $value | quote}}
    {{- end }}
  ```

  - Step 5 : Create Secret

    - I also use Range to Loop the List of Secret Data
  
  ```
  apiVersion: v1 
  kind: Secret
  metadata:
    name: {{ .Values.secretName }}
  type: Opaque
  data: 
    {{- range $key, $value := .Values.secretData }}
    {{ $key }}: {{ $value }}
    {{- end }}
  ```

  - Step 6 : Create Ingress

  ```
  apiVersion: networking.k8s.io/v1
  kind: Ingress 
  metadata: 
    name: {{ .Values.ingressName }}
  spec:
    ingressClassName: nginx 
    rules: 
    - host: {{ .Values.ingressHost }}
      http: 
        paths: 
        - path: / 
          pathType: Prefix
          backend:
            service:
              name: {{ .Values.appName }}
              port:
                number: {{ .Values.servicePort }}
  ```




