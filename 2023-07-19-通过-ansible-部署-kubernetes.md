---
title: 通过 Ansible 部署 kubernetes
thumbnailImagePosition: left
thumbnailImage: https://img.freepik.com/premium-photo/manager-industrial-engineer-using-tablet-check-control-automation-robot-arms-machine_34200-348.jpg
date: 2023-07-18
categories:
- Kubernetes
- Ansible
tags:
- Kubernetes
- Ansible
---

前面我们已经演示过，如何使用二进制的方式部署一个 kubernetes 群集，今天我们来看一下看，如果通过 Ansible 来实现这个过程。

<!--more-->

该项目目前只支持 Ubuntu。

准备环境：  
TestMaster 192.168.3.190  
TestNode1 192,168,3,191  
TestNode2 192.168.3.192

从 GitHub 下载项目文件
```bash
root@ansible:~# git clone https://github.com/Aaron-Yu1/kubelazy.git
root@ansible:~# cd kubelazy/

# 创建 bin 目录，用于存放下载文件
root@ansible:~/kubelazy# mkdir bin
root@ansible:~/kubelazy# cd bin/
```

下载软件包。如果觉得下载的慢的，可以通过 GitHub proxy 来下载
```bash
root@ansible:~/kubelazy/bin# wget https://github.com/cloudflare/cfssl/releases/download/v1.6.3/cfssljson_1.6.3_linux_amd64 -O cfssljson
root@ansible:~/kubelazy/bin# wget https://github.com/cloudflare/cfssl/releases/download/v1.6.3/cfssl_1.6.3_linux_amd64 -O cfssl
root@ansible:~/kubelazy/bin# chmod +x cfss*
root@ansible:~/kubelazy/bin# ls -l
total 20496
-rwxr-xr-x 1 root root 13336752 Oct  4  2022 cfssl
-rwxr-xr-x 1 root root  7646320 Oct  4  2022 cfssljson

# 下载 etcd 并解压
root@ansible:~/kubelazy/bin# wget https://github.com/etcd-io/etcd/releases/download/v3.5.6/etcd-v3.5.6-linux-amd64.tar.gz
root@ansible:~/kubelazy/bin# tar -zxf etcd-v3.5.6-linux-amd64.tar.gz
root@ansible:~/kubelazy/bin# ls
cfssl  cfssljson  etcd-v3.5.6-linux-amd64  etcd-v3.5.6-linux-amd64.tar.gz

下载 kubernetes 二进制包，并解压
root@ansible:~/kubelazy/bin# wget https://dl.k8s.io/v1.26.4/kubernetes-server-linux-amd64.tar.gz
root@ansible:~/kubelazy/bin# tar -zxf kubernetes-server-linux-amd64.tar.gz

下载 runtime，并解压（目前只支持 containerd）
root@ansible:~/kubelazy/bin# wget https://github.com/containerd/containerd/releases/download/v1.6.19/cri-containerd-cni-1.6.19-linux-amd64.tar.gz
root@ansible:~/kubelazy/bin# tar -zxf cri-containerd-cni-1.6.19-linux-amd64.tar.gz
```


更改 hosts 文件：
```bash
root@ansible:~/kubelazy/bin# cd ..
root@ansible:~/kubelazy# vim inventory/hosts 
root@ansible:~/kubelazy# cat inventory/hosts
# This is the default 'hosts' file.
#
# You should change it and replace IP address
# And change the node name to your won node name
[master]
192.168.3.190

# ETCD_NODE_NAME 是 etcd 节点的名称。
[etcd]
192.168.3.190 ETCD_NODE_NAME=testmaster
192.168.3.191 ETCD_NODE_NAME=testnode1
192.168.3.192 ETCD_NODE_NAME=testnode2

# K8S_NODE_NAME 是k8s 节点的名称。
[work]
192.168.3.191 K8S_NODE_NAME=testnode1
192.168.3.192 K8S_NODE_NAME=testnode2

[ha]
#192.168.3.221 STATE=MASTER PRIORITY=100
#192.168.3.222 STATE=BACKUP PRIORITY=90

# 基于需要，选择是否安装 harbor，如果需要，更改到对应的地址，并取消注释。
[harbor]
#192.168.3.193
```

配置 ssh 免密登录
```bash
root@ansible:~/kubelazy# ssh-copy-id 192.168.3.190
root@ansible:~/kubelazy# ssh-copy-id 192.168.3.191
root@ansible:~/kubelazy# ssh-copy-id 192.168.3.192
```

运行 ansible playbook
```bash
root@ansible:~/kubelazy# ansible-playbook kubelazy.yml
```