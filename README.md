# How Install WordPress on Kubernetes
This article is for who are trying to install worpress using kubernets
---

It's only for demo purpose, here I am using minikube cluster in ubuntu ec2 machine so if you are not familiar with installing minikube, please refer this and start...

Step 1 : Creating namespace

```
apiVersion: v1
kind: Namespace
metadata:
  name: demo-wordpress

```


