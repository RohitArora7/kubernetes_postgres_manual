# kubernetes_postgres_manual

Create Config map
```bash
nano postgres-configmap.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: postgres-config
  labels:
    app: postgres
data:
  POSTGRES_DB: postgresdb
  POSTGRES_USER: admin
  POSTGRES_PASSWORD: test123
  
  
kubectl apply -f postgres-configmap.yaml

```
___________________________________________


Create and Apply Persistent Storage Volume and Persistent Volume Claim

```bash
nano postgres-storage.yaml

kind: PersistentVolume
apiVersion: v1
metadata:
  name: postgres-pv-volume
  labels:
    type: local
    app: postgres
spec:
  storageClassName: manual
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: "/mnt/data"
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: postgres-pv-claim
  labels:
    app: postgres
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
      
      
kubectl apply -f postgres-storage.yaml


```
______________________________________________________________________________


Create and Apply PostgreSQL Deployment

```bash
nano postgres-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:10.1
          imagePullPolicy: "IfNotPresent"
          ports:
            - containerPort: 5432
          envFrom:
            - configMapRef:
                name: postgres-config
          volumeMounts:
            - mountPath: /var/lib/postgresql/data
              name: postgredb
      volumes:
        - name: postgredb
          persistentVolumeClaim:
            claimName: postgres-pv-claim


kubectl apply -f postgres-deployment.yaml

```
______________________________________________________________________________


Create and Apply PostgreSQL Service

```bash
nano postgres-service.yaml

apiVersion: v1
kind: Service
metadata:
  name: postgres
  labels:
    app: postgres
spec:
  type: NodePort
  ports:
   - port: 5432
  selector:
   app: postgres
   
kubectl apply -f postgres-service.yaml

```

______________________________________________________________________________

Connect
```bash
kubectl exec -it [pod-name] --  psql -h localhost -U admin --password -p [port] postgresdb


kubectl exec -it postgre --  psql -h localhost -U admin --password -p 5432 postgresdb

```


      
        
