---
title: 使用二进制方式部署 kubernetes(5)：部署网络插件
thumbnailImagePosition: left
thumbnailImage: https://img.freepik.com/premium-photo/cloud-computer-technology-storage-online-computer-business-network-ideas-connected-internet-server-services-cloud-transfer-shown-future-network-data_29488-8454.jpg
date: 2023-05-18
categories:
- Kubernetes
- calico
tags:
- calico
- kubernetes
- kubernetes network
---
在这个示例中，我们使用的 calico。

<!--more-->

在选择版本之前，我们应该先查看指定版本的 [requirements](https://docs.tigera.io/calico/3.25/getting-started/kubernetes/requirements) 页面，以确定 calico 的版本是否支持当前的 kubernetes 版本。

calico 支持两种方式安装

  * Operator
  * Manifest

我们以 Manifest 为例。下载对应的 yaml 文件，并更改 CALICO_IPV4POOL_CIDR 的值
```bash
root@master:~# curl https://raw.githubusercontent.com/projectcalico/calico/v3.25.1/manifests/calico.yaml -O
rootroot@master:~# cat calico.yaml | grep CALICO_IPV4POOL_CIDR -A 1
            - name: CALICO_IPV4POOL_CIDR
              value: "172.16.0.0/16"

# 创建对应的 pod
root@master:~# kubectl apply -f calico.yaml

# 验证
root@master:~# kubectl get pods -A
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-5857bf8d58-6jpbb   1/1     Running   0          34s
kube-system   calico-node-hg8r4                          1/1     Running   0          34s
kube-system   calico-node-l2vc9                          1/1     Running   0          34s
```