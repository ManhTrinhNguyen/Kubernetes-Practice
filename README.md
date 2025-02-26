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
    












