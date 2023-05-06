# How Install WordPress on Kubernetes
This article is for who are trying to install worpress using kubernets
---

It's only for demo purpose, here I am using minikube cluster in ubuntu ec2 machine so if you are not familiar with installing minikube, please refer this and start...

## Step 1 : Creating namespace

```sh
apiVersion: v1
kind: Namespace
metadata:
  name: demo-wordpress
  
```

I have used demo-wordpress as namespace of this project

## step 2 : Creating PersistentVolume and persistentVolumeClaim for both mysql and wordpress

```
kind: PersistentVolume
metadata:
  name: demo-mysql-pv
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: "/mnt/demo-mysql-data"
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: demo-wordpress-pv
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: "/mnt/demo-wordpress-pv"
```
