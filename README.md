# How Install WordPress on Kubernetes
This article is for who are trying to install worpress using kubernets
---

It's only for demo purpose, here I am using minikube cluster in ubuntu ec2 machine so if you are not familiar with installing minikube, please refer this and start...

### Step 1 : Creating namespace

```sh
apiVersion: v1
kind: Namespace
metadata:
  name: demo-wordpress
  
```

I have used demo-wordpress as namespace of this project

### step 2 : Creating PersistentVolume both mysql and wordpress

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
### step 3 : Creating persistentVolumeClaim for both mysql and wordpress

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
### step 4 : Create a secret to store the mysql password

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

### Step 4: Mysql deploy

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-mysql
  namespace: demo-wordpress
  labels:
    app: demo-mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: demo-mysql
  template:
    metadata:
      labels:
        app: demo-mysql
    spec:
      containers:
        - name: demo-mysql-container
          image: mysql:8
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 3306
          resources:
            limits:
              memory: '1Gi'
              cpu: '1'
          volumeMounts:
            - name: demo-mysql-data
              mountPath: "/var/lib/mysql"
          env:
            - name: MYSQL_DATABASE
              value: wordpress
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: wp-secrets
                  key: MYSQL_ROOT_PASSWORD
      volumes:
        - name: demo-mysql-init
          configMap:
             name: demo-mysql-wp-config
        - name: demo-mysql-data
          persistentVolumeClaim:
            claimName: demo-mysql-pvc


---

apiVersion: v1
kind: ConfigMap
metadata:
  name: demo-mysql-wp-config
  namespace: demo-wordpress
data:
  init.sql: |
     CREATE DATABASE IF NOT EXISTS wordpress;
```

### step 5: Msyql TCP service creating

```
apiVersion: v1
kind: Service
metadata:
  name: demo-wordpress-svc
  namespace: demo-wordpress
spec:
  selector:
    app: demo-mysql
  ports:
    - protocol: TCP
      port: 3306
```
### step 6: Wordpress deploy
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-wordpress
  namespace: demo-wordpress
  labels:
    app: demo-wordpress
spec:
  replicas: 1
  selector:
    matchLabels:
      app: demo-wordpress
  template:
    metadata:
      labels:
        app: demo-wordpress
    spec:
      containers:
        - name: demo-wordpress-container
          image: wordpress:6.1.1
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 80
          volumeMounts:
            - name: wordpress-data
              mountPath: /var/www/html/wp-content
          env:
            - name: WORDPRESS_DB_HOST
              value: demo-wordpress-svc:3306
            - name: WORDPRESS_DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: wp-secrets
                  key: MYSQL_ROOT_PASSWORD
            - name: WORDPRESS_DB_USER
              value: root
            - name: WORDPRESS_DB_NAME
              value: wordpress
          lifecycle:
            postStart:
              exec:
                command: ["/bin/bash", -c , "chown -R www-data:www-data /var/www/; chmod -R 774 /var/www"]
      volumes:
        - name: wordpress-data
          persistentVolumeClaim:
            claimName: demo-wordpress-pvc
```
### Step 7 : Create the svc for the mysql
```
apiVersion: v1
kind: Service
metadata:
  name: demo-wordpress-service
  namespace: demo-wordpress
  labels:
    app: demo-wordpress
spec:
  ports:
    - port: 80
  selector:
    app: demo-wordpress
  type: LoadBalancer
```
### Step 8: If you are using the minikube then the loadbalcer type service always showing pending, check the below part...

```
root@ip-172-31-44-197:~/wordpress# kubectl get svc -n demo-wordpress
NAME                     TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
demo-wordpress-service   LoadBalancer   10.111.208.233   <pending>     80:31356/TCP   135m
demo-wordpress-svc       ClusterIP      10.111.99.7      <none>        3306/TCP       3h13m
```
### So you need to run another command for geting the loadbalance url

```
root@ip-172-31-44-197:~/wordpress# minikube service -n demo-wordpress demo-wordpress-service --url
http://192.168.49.2:31356
```
### So in our case I got one url http://192.168.49.2:31356 so I need to do ssh tunnel to the local machine for connecting the wordpress url from my local machine.

Ssh tunnel command is the following

```
ssh -i [private key.pem] -L [local port]:localhost:[remote port] username@[public ip]
```
### So open another terminal and run the following command.

ssh -i privatekey.pem -L 8080:192.168.49.2:31356 ubuntu@13.235.246.74

### So you are all set, you can access the wordpress from your local machine using the following url.

```
https://localhost:8080
```

![](https://i.ibb.co/RyxTmSn/image.png)

