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

## step 2 : Creating PersistentVolume both mysql and wordpress

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
## step 3 : Creating persistentVolumeClaim for both mysql and wordpress

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: demo-mysql-pvc
  namespace: demo-wordpress
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 100Mi
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: demo-wordpress-pvc
  namespace: demo-wordpress
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```
## step 4 : Create a secret to store the mysql password

So we need to base64 a password for the root user. I used the following random password for the root user, the following is the my root password.

fmPw421TSfI82LYGeM

then I run this command to get the base64 version of it:
```
 echo -n "fmPw421TSfI82LYGeM" | base64
 Zm1QdzQyMVRTZkk4MkxZR2VN
```
---
```
apiVersion: v1
kind: Secret
metadata:
  name: wp-secrets
  namespace: demo-wordpress
type: Opaque
data:
  MYSQL_ROOT_PASSWORD: Zm1QdzQyMVRTZkk4MkxZR2VN
```

