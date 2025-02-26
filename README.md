# ConfigMap and Volume Type in K8 

- ConfigMap and Secret are used for External Configuration of individual Value

- Secret is used to store Sensitive Data

- Configmap is used to store Non-Sensitive Data

**Configmap Example**
```
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: config-file
  data: 
    database_url: mongodb-service # This is a key-value pair 
```

**Secret Example**
```
  apiVersion: v1
  kind: Secret
  metadata:
    name: Secret-file
  data: 
    username : <value in base64> # This is a key-value pair
    password : <Value in base64> # This is a key-value pair

!!! Note: Value in Secret is Base62 encode 
```

**Deployment/Pod references key-value from Secret and ConfigMap**
```
env:
- name: MONGO_INITDB_ROOT_USERNAME
  valueFrom:
    secretKeyRef: # Reference Key-Value from Secret
      name: Secret-file
      key: username
- name: MONGO_INITDB_ROOT_PASSWORD
  valueFrom:
    secretKeyRef: # Reference Key-Value from Secret
      name: Secret-file
      key: password
- name: MONGO_DB_SERVER
  valuefrom:
    configMapKeyRef: # Reference Config-map From ConfigMap
      name: config-file
      key: database_url
```

**Create File and then mount into Pod and then into Container**

**ConfigMap File Example**
```
----I have the same Configmap componet----

apiVersion: v1
kind: ConfigMap
metadata:
  name: mosquitto-config-file
data:
  # Instead of define Key-value pair . I create name of the File and Content of the file 
  mosquitto.conf: |  # Name of the file 
      log_dest stdout # Content of the file 
      log_type all # Content of the file
      log_timestamp true # Content of the file
      listener 9001 # Content of the file
```

**Secret File Example**
```
----I have the same Secret componet----

apiVersion: v1
kind: Secret
metadata:
  name: secret-file
data:
  # Instead of define Key-value pair . I create name of the File and Content of the file 
  secret.file: |  # Name of the file 
      cGFzc3dvcmQ= # Value of the file in base64 encoded
```

## Mosquitto Application Demo 

Step 1 : Create ConfigMap and Secret for external Configuration 

  ----ConfigMap for Mosquitto----
  - What Port to Open ?
  - Wheather secure or not ?

  ----Secret for Mosquitto----
  - Password for authentication
  - Or overwrite the Certificate file for Mosquitto
    

Step 2 : Create Mosquitto Deployment without Volume (To see default structure)

  - To get into the container : `kubectl exec -it <podname> -- bin/sh`

  - Get into Mosquitto config : `cd /mosquitto/config | cat mosquitto.conf`

  !!! After I can see the default (for learning purpose) I will delete that deployment 

Step 3 : Create Misquitto Deployment with Volume

  - I will overwrite Mosquitto Config File using the ConfigMap by mounting it into the Container

!!! Note: ConfigMap and Secret must exist before Pod start

  ----To mount Volume into the Pod----
  ```
    containers:
    - name: mosquitto
      image: eclipse-mosquitto:2.0
      ports:
        - containerPort: 1883
    volumes: # This is a list of volumes that I want to attach to the pod
      - name: mosquitto-config # this is a name of the volume
        configMap: # This is a type of the Volume I want to mount into the Pod
          name: mosquitto-config-file # This is the name of the ConfigMap that I want to mount
       
      - name: mosquitto-secret # the same for secret  
        secret:
          secretName: mosquitto-secret-file
  ```

  ----Now I have volume in the Pod . I will mount it into the Container . Bcs Application run inside the Container----

  - Inside Each individual Container:

  ```
  containers:
    - name: mosquitto
      image: eclipse-mosquitto:2.0
      ports:
        - containerPort: 1883

      volumeMounts: # This is a list of volumes that I want to mount from Pod into the container
        - name: mosquitto-config # This is the name of the volume
          mountPath: /mosquitto/config # This is the path where I want to mount the volume
        - name: mosquitto-secret
          mountPath: /mosquitto/secret # If secret not exists, it will be created
          readOnly: true # This is a read-only volume . Need for Certificates bcs i don't want to change them

  volumes: # This is a list of volumes that I want to attach to the pod
    - name: mosquitto-config # this is a name of the volume
      configMap: # This is a type of the Volume I want to mount into the Pod
        name: mosquitto-config-file # This is the name of the ConfigMap that I want to mount
     
    - name: mosquitto-secret # the same for secret  
      secret:
        secretName: mosquitto-secret-file
  ```

  - The Path value depend on the Application

  - The concept of Mounting to the Pod then Mounting to the Container is useful for if I have multiple Container , I can decide which container get access to Which volumes that Pod has available

Step 4 : After execute Mosquitto yaml with Volume 

  - To check inside container : `kubectl exec -it <podname> -- /bin/sh`

  - In `/mosquitto/config` I will see Secrect file haven't been there before with the value I provided in Secret

  - Also in `/mosquitto/config/mosquitto.conf` get overwrited by my ConfigMap


**Wrap up**

- Both ConfigMap and Secrect can be used as Key-Value pair for the invidual ENV

- Or I can Create File For them that I can pass in as Configuration File

!!! Note : ConfigMap and Secret are Local Volume Type 






