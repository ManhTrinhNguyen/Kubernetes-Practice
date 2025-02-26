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
    username : <value in base62> # This is a key-value pair
    password : <Value in base62> # This is a key-value pair

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



















