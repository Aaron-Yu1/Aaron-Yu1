---
title: 使用二进制方式部署 kubernetes(6)：部署 DNS
thumbnailImagePosition: left
thumbnailImage: https://img.freepik.com/premium-photo/social-media-communication-internet-network-connection-city_43780-6092.jpg
date: 2023-05-18
categories:
- Kubernetes
- CoreDNS
tags:
- Kubernetes
- CoreDNS
- DNS
- kubernetes DNS
---
在这个示例中，我们使用 DNS 是 CoreDNS。

<!--more-->

从 GitHub 下载对应的源码包，并更改 deploy.sh 脚本，更改 “CLUSTER_DOMAIN” 和 CLUSTER_DNS_IP”。
```bash
root@testmaster:~# apt-get install -y jq
root@testmaster:~# git clone https://github.com/coredns/deployment.git
root@testmaster:~# vim deployment/kubernetes/deploy.sh
root@testmaster:~# cat deployment/kubernetes/deploy.sh | grep -e CLUSTER_DNS_IP=1 -e cluster.local
CLUSTER_DOMAIN=cluster.local
  CLUSTER_DNS_IP=192.168.0.2
```
这里的 DNS 地址要和 kubelet.config 文件中指定的一致

通过 deploy.sh 脚本生成 yaml 文件，并使用此 yaml 文件创建 pod；
```bash
root@testmaster:~# deployment/kubernetes/deploy.sh > coredns.yaml
root@testmaster:~# ls
calico.yaml   deployment    etcd-v3.5.5-linux-amd64         kubernetes                            master_ssl.cnf
coredns.yaml  etcd_ssl.cnf  etcd-v3.5.5-linux-amd64.tar.gz  kubernetes-server-linux-amd64.tar.gz  snap
root@testmaster:~# kubectl apply -f coredns.yaml

root@master:~# kubectl get pods -A
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-5857bf8d58-6jpbb   1/1     Running   0          12m
kube-system   calico-node-hg8r4                          1/1     Running   0          12m
kube-system   calico-node-l2vc9                          1/1     Running   0          12m
kube-system   coredns-7f5598cff-r82tk                    1/1     Running   0          53s

root@master:~# kubectl get service -n kube-system
NAME       TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   192.168.0.2   <none>        53/UDP,53/TCP,9153/TCP   12d
```