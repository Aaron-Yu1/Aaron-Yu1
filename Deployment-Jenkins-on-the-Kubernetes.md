---
title: "在 Kubernetes 上部署 Jenkins"
thumbnailImagePosition: left
thumbnailImage: https://img.freepik.com/free-photo/rpa-concept-with-blurry-hand-touching-screen_23-2149311914.jpg
coverImage: https://img.freepik.com/free-photo/hands-typing-laptop-top-view_23-2149311919.jpg
metaAlignment: center
coverMeta: out
date: 2015-05-13
categories:
- Jenkins
- CI/CD
- Kubernetes
tags:
- Jenkins
- CI/CD
- Kubernetes
---

准备安装使用的 yaml 文件。

<!--more-->

```bash
root@master:~# kubectl create namespace jenkins
root@master:~# vim jenkins.yaml
root@master:~# cat jenkins.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      containers:
      - name: jenkins
        image: jenkins/jenkins:lts
        ports:
          - name: http-port
            containerPort: 8080
          - name: jnlp-port
            containerPort: 50000
        volumeMounts:
          - name: jenkins-vol
            mountPath: /var/jenkins_vol
      volumes:
        - name: jenkins-vol
          emptyDir: {}

root@master:~# kubectl apply -f jenkins.yaml --namespace jenkins
deployment.apps/jenkins created
root@master:~# kubectl get pods -n jenkins
NAME                       READY   STATUS              RESTARTS   AGE
jenkins-655f6c69dd-8kstg   0/1     ContainerCreating   0          30s
root@master:~# kubectl get pods -n jenkins
NAME                       READY   STATUS    RESTARTS   AGE
jenkins-655f6c69dd-8kstg   1/1     Running   0          2m9s

```

```bash
root@master:~# vim jenkins-service.yaml
root@master:~# cat jenkins-service.yaml 
apiVersion: v1
kind: Service
metadata:
  name: jenkins
spec:
  type: NodePort
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 30008
  selector:
    app: jenkins

---

apiVersion: v1
kind: Service
metadata:
  name: jenkins-jnlp
spec:
  type: ClusterIP
  ports:
    - port: 50000
      targetPort: 50000
  selector:
    app: jenkins

root@master:~# kubectl apply -f jenkins-service.yaml --namespace jenkins
service/jenkins created
service/jenkins-jnlp created

root@master:~# kubectl get services --namespace jenkins
NAME           TYPE        CLUSTER-IP        EXTERNAL-IP   PORT(S)          AGE
jenkins        NodePort    192.168.150.112   <none>        8080:30008/TCP   72s
jenkins-jnlp   ClusterIP   192.168.217.199   <none>        50000/TCP        72s

```



```bash
root@master:~# kubectl logs jenkins-655f6c69dd-8kstg -n jenkins
Running from: /usr/share/jenkins/jenkins.war
webroot: /var/jenkins_home/war
2023-08-21 06:48:35.718+0000 [id=1]	INFO	winstone.Logger#logInternal: Beginning extraction from war file
2023-08-21 06:48:37.506+0000 [id=1]	WARNING	o.e.j.s.handler.ContextHandler#setContextPath: Empty contextPath
... ...
... ...
2023-08-21 06:48:41.798+0000 [id=30]	INFO	jenkins.install.SetupWizard#init: 

*************************************************************
*************************************************************
*************************************************************

Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

*********<Password>*********

This may also be found at: /var/jenkins_home/secrets/initialAdminPassword

*************************************************************
*************************************************************
*************************************************************

2023-08-21 06:49:03.178+0000 [id=30]	INFO	jenkins.InitReactorRunner$1#onAttained: Completed initialization
2023-08-21 06:49:03.196+0000 [id=22]	INFO	hudson.lifecycle.Lifecycle#onReady: Jenkins is fully up and running
2023-08-21 06:50:06.024+0000 [id=44]	INFO	h.m.DownloadService$Downloadable#load: Obtained the updated data file for hudson.tasks.Maven.MavenInstaller
2023-08-21 06:50:06.025+0000 [id=44]	INFO	hudson.util.Retrier#start: Performed the action check updates server successfully at the attempt #1

```