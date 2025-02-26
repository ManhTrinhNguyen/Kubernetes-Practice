# ConfigMap and Volume Type in K8 

- ConfigMap and Secret are used for External Configuration of individual Value

- Secret is used to store Sensitive Data

- Configmap is used to store Non-Sensitive Data

**Configmap Example**
```
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: mosquitto-config-file
  data: 
    database_url: mongodb-service # This is a key-value pair 
```

**Secret Example**
```
  apiVersion: v1
  kind: Secret
  metadata:
    name: Secret
  data: 
    username : <value in base62>
    password : <Value in base62>

!!! Note: Value in Secret is Base62 encode 
```
