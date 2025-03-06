# Create HelmChart for Microservices 

**What is Helm Chart**

```
  - Helm Chart is a bundle of a bunch of Yaml file .

  - When I deploy an App with those Configuration yaml file I can bundle it for the next use . I don't have to do Configuration again
```

**3 Ways to create Helm Chart***

```
  1. Create separate Helm Chart when Configuration is very different

  2. Create 1 Helm Chart when Configurations is very the same

  3. Combination of both Options
```

**Basic Structure of HelmChart**

- Create HelmChart `helm create <helm-chart-name>`

- Inside Helm Chart Folder :

  - **chart.yaml** : Describe metadata
 
  - **Chart Folder**: Chart Folder hold any dependencies charts that these charts might have
 
  - **.helmignore**:
    - File don't want to include in my Helm Chart
    - Basically this file tell Helm to ignore all of these Folder and Files when building a Package Archive from this chart . Helm Chart can build and packaged and distributed just like any other artifacts
   
  - **Template Folder**:
    - This is a core Helm Chart
   
    - This is where actual K8s Yaml file created
   
    - .yaml file : The attribute same as in the K8s Service configuration file . Instead of hardcode value I have placeholder defined as {{}} the value of this syntax are actually the placeholder to actual Value
   
  - **value.yaml**
    - This is a place where the acutal value are set will be then substitued in the template file

**Overview**

  - Step 1 : I create Helmchart for all Microservices and Redis

  ```
    -- Syntax to create Helm Chart: `helm create <helmchart-name>` 
    -- Redis have to be separate bcs Redis has different configuration
  ```
 
  - Step 2 : Cofigure Deployment and Service in those Helm Chart

  ```
    -- Instead of hardcode Value I use this syntax for dynamically Values : `{{ .Values.<name-of-the-value>}}`
    
    -- This syntax is to configure LivenessProbe and ReadinessProbe or anything need correct Yaml Format : `{{- toYaml
.Values.livenessProbe | nindent 12 }}`

      --- toYaml : Ensure the LivenessProbe or RedinessProbe is properly formatted instead of being rendered as a single-line string
      --- nindent 12 : Helm Template require correct indention inside deployment.yaml , and `nindent 12` correctly format it at 12 spaces 
  ```

  - Step 3 : Set Values for those Deployment and Serivce . Also for Redis

  ```
    - In Values file : Set value to <name-of-the-value> 
  ```

  - Step 4 : Validate Template File

    -- Using: `helm template -f <value-file> <name-of-helm-chart>` | Give me nice preview from all the values sources
    
    -- Another the way to Validate values yamle file : `helm lint <value-file> <chart-name>`

  - Step 5 : Deploy Helm Chart

    -- `helm install -f <value file> <release-name> <chart name> --namespace microservices` 
    
    -- To delete : `helm delete <release-name>`

  - Step 6 : Create Helmfile

  ```
    - Helmfile is a tool that helps me manage and deploy multiple Helm charts easily.
  
    - I Can define multiple release and then change specification of each release depending on Application itself 
    
    
      -- Install Helmfile : brew install helmfile
  
      -- helm file sync :
      
        --- Prepare all the Release 
        --- It will then compare the acutal State in the Cluster with the desired state that I configured in Helmfile
        --- Base on that It will plan what need to be install and deployed in the Cluster to give me a desire State 
  
      -- helm list : Which show me at any point the currently installed released that Helmfile manage for me 
        
      - Helmfile Configuration:
  
          releases:
          - name: my-app
            namespace: default
            chart: stable/nginx
            version: 1.2.3
            values:
  ```

  - Step 7 : Deploy Ingress Controller by using Helm

  ```
    - Ingress Controller evaluate all the Rules in Ingress Component . That way to manage all the redirection . This will be an Entry Point for my Cluster

    Step 1 : Add NGINX repo : `helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx`

    Step 2 : Installing the helm chart : `helm install ingress-controller ingress-nginx/ingress-nginx --set controller.publishService.enabled=true`

      -- --set controller.publishService.enabled=true: This attr make sure it automatically allocated the Public IP address

    - Ingress Controller  use Cloud Native in the background . Linode's own NodeBalancer dynamically created and provisioned as I created Ingress Controller

     - This NodeBalancer is an Entry Point of my K8s Cluster -> This NodeBalancer give me an External IP, Ports -> and NodeBalancer will forward the request coming in into Cluster to the Ingress Controller and to the Service base on Ingress rule I create
  ```

  - Step 8 : Deploy Ingress Rule
    
**Deployment**

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.appName }}
  labels:
    app: {{ .Values.appName }}
spec: 
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Values.appName }}
    template:
      metadata:
        labels:
          app: {{ .Values.appName }}
          spec: 
            containers:
            - name: {{ .Values.appName }}
              image: {{ .Values.appImage }}:{{ .Values.appVersion }}
              ports:
              - containerPort: {{ .Values.containerPort }}
              livenessProbe:
                {{ -toYaml .Values.livenessProbe | nindent 12 }} # toYaml is a function that converts the livenessProbe to yaml format
              readinessProbe:
                {{.Values.readinessProbe }}
              resources:
                requests: 
                  memory: {{ .Values.memoryRequest }}
                  cpu: {{ .Values.cpuRequest }}
                limits:
                  memory: {{ .Values.memoryLimit }}
                  cpu: {{ .Values.cpuLimit }}
              env: 
              {{ -range .Values.containerEnvVars }}
              - name: {{ .key }}
                value: {{ .value | quote }}
              {{ -end }}                

```

**Service**

```
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.appName }}
  labels:
    app: {{ .Values.appName }}
spec:
  selector:
    app: {{ .Values.appName }}
  ports:
    - protocol: TCP
      port: {{ .Values.servicePort}}
      targetPort: {{ .Values.containerPort }}
      type: {{ .Values.serviceType }}
```

**Dynamic ENV**

- For one single ENV I can have container ENV var like this :
  
```
env: 
- name: {{ .Values.containerEnvVar.key }}
  value: {{ .Values.containerEnvVar.value }}
```

- For working with List of something . In this case ENV, Helm has built a function called Range

- Range is basically loop through a List and give me element one by one

- Syntax for Range . This way I can access the key value pair that I define in the Container Env var list

```
env:
{{-range .Values.containerEnvVars}}
- name: {{ .key }}
  value: {{ .value | quote }}
{{-end}}

Note: Value ENV variable alway interpreted as strings . So I use a built-in function called quote and will use piping syntax |

- Piping syntax in Linux : is the way to pass Output of 1 command to other Input of other command
```

**Create values file Example (Default Value in the Chart)**

```
appName: emailservice
replicasCount: 2
appImage: gcr.io/google-samples/microservices-demo/emailservice
appVersion: v0.8.0
containerPort: 8080
servicePort: 8080
serviceType: ClusterIP
containerEnvVars: 
  - name: PORT
    value: "8080"
  - name: DISABLE_PROFILER
    value: "1"
livenessProbe:
  grpc:
    port: 8080
  periodSeconds: 5

readinessProbe:
  grpc:
    port: 8080
  periodSeconds: 5
memoryRequest: 64Mi
cpuRequest: 250m
memoryLimit: 128Mi
cpuLimit: 500m
```

- Then I will create value for all Micro-services
- 

























  
