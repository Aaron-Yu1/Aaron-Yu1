---
title: 使用二进制方式部署 kubernetes(3)：部署 Master 节点
thumbnailImagePosition: left
thumbnailImage: https://img.freepik.com/premium-photo/lock-laptop-background-cyber-safety-concept_488220-1737.jpg
date: 2023-05-18
categories:
- kubernetes
tags:
- kube-apiserver
- kube-controller-manager
- kube-scheduler
- kubernetes
- master
---

前面，我们已经准备好了 etcd，现在我们来看一看，如何通过二进制的方式部署 master 节点。

<!--more-->

下载二进制包，并解压，然后复制 kube-apiserver, kube-controller-manager, kube-scheduler 以及 kubectl 到 /usr/local/bin/。
```bash
root@master:~/certs# cd ../pkg/
root@master:~/pkg# wget https://dl.k8s.io/v1.26.4/kubernetes-server-linux-amd64.tar.gz
root@master:~/pkg# tar -zxf kubernetes-server-linux-amd64.tar.gz

# 复制文件
root@master:~/pkg# cp kubernetes/server/bin/{kube-apiserver,kube-controller-manager,kube-scheduler,kubectl} /usr/local/bin/
```


### 生成 .kube/config 文件
创建证书请求
```powershell
root@master:~/pkg# cd ../certs/
root@master:~/certs# vim admin-csr.json
root@master:~/certs# cat admin-csr.json
{
    "CN": "admin",
    "hosts": [],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "JS",
            "L": "SZ",
            "O": "system:masters",
            "OU": "System"
        }
    ]
}
```
因为该证书是作为用户验证用得，所以 hosts 的值可以为空；  
Kubernetes 在生成配置文件的时候，会默认将 "CN" 的值作为用户名，"O" 字段的值作为组。所以在创建证书请求的时候，将 "CN" 指定为 “admin”，将 "O" 指定为 “system:masters”。  
默认 ClusterRoleBinding 将 system:masters 与 cluster-admin 绑定，所以将 admin 加入到 system:masters，就拥有了群集的所有权限了。  
我们可以在已经安装好的群集中通过 kubectl describe clusterrolebinding cluster-admin 命令来查看。

使用前面生成的证书请求文件创建 admin 证书
```bash
root@master:~/certs# cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes admin-csr.json | cfssljson -bare admin
2023/05/05 10:26:47 [INFO] generate received request
2023/05/05 10:26:47 [INFO] received CSR
2023/05/05 10:26:47 [INFO] generating key: rsa-2048
2023/05/05 10:26:47 [INFO] encoded CSR
2023/05/05 10:26:47 [INFO] signed certificate with serial number 3636844409725463248947852867506746610085159226
2023/05/05 10:26:47 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
websites. For more information see the Baseline Requirements for the Issuance and Management
of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
specifically, section 10.2.3 ("Information Requirements").
root@master:~/certs# ls -l admin*
-rw-r--r-- 1 root root  997 May  5 10:26 admin.csr
-rw-r--r-- 1 root root  277 May  5 10:26 admin-csr.json
-rw------- 1 root root 1679 May  5 10:26 admin-key.pem
-rw-r--r-- 1 root root 1363 May  5 10:26 admin.pem
```

使用 kubectl 命令生成 conf 文件
```bash
root@master:~/certs# kubectl config set-cluster kubernetes --certificate-authority=ca.pem --embed-certs=true --server=https://10.0.0.99:6443
Cluster "kubernetes" set.
root@master:~/certs# kubectl config set-credentials admin --client-certificate=admin.pem --embed-certs=true --client-key=admin-key.pem
User "admin" set.
root@master:~/certs# kubectl config set-context kubernetes --cluster=kubernetes --user=admin
Context "kubernetes" created.
root@master:~/certs# kubectl config use-context kubernetes
Switched to context "kubernetes".
```
--server 用于指定 kube-apiserver 地址

在当前用户的 home 目录下创建了 .kube/conf 文件
```bash
root@master:~/certs# ls -l ~/.kube/config 
-rw------- 1 root root 6085 May  5 10:44 /root/.kube/config
```

### 配置 kube-apiserver
创建 kube-apiserver 证书请求
```bash
# 在 hosts 中，除了添加 kube-apiserver 的 IP 地址，还需要添加 kubernetes 的 service 地址
# 在多 kube-apiserver 的情况下，还需要添加 LB 的 VIP 地址
# cluster.local 为 DNS 域名为 
root@master:~/certs# vim server-csr.json 
root@master:~/certs# cat server-csr.json
{
  "CN": "kubernetes",
  "hosts": [
    "127.0.0.1",
    "10.0.0.99",
    "192.168.0.1",
    "kubernetes",
    "kubernetes.default",
    "kubernetes.default.svc",
    "kubernetes.default.svc.cluster",
    "kubernetes.default.svc.cluster.local"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "JS",
      "L": "SZ",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
```

使用前面的证书请求文件，生成证书
```bash
root@master:~/certs# cfssl gencert -ca=ca.pem -ca-key=ca-key.pem --config=ca-config.json -profile=kubernetes server-csr.json | cfssljson -bare server
2023/05/05 10:53:38 [INFO] generate received request
2023/05/05 10:53:38 [INFO] received CSR
2023/05/05 10:53:38 [INFO] generating key: rsa-2048
2023/05/05 10:53:38 [INFO] encoded CSR
2023/05/05 10:53:38 [INFO] signed certificate with serial number 187534172965388516038194123547724889787341674166
root@master:~/certs# ls -l server*
-rw-r--r-- 1 root root 1220 May  5 10:53 server.csr
-rw-r--r-- 1 root root  405 May  5 10:52 server-csr.json
-rw------- 1 root root 1675 May  5 10:53 server-key.pem
-rw-r--r-- 1 root root 1566 May  5 10:53 server.pem
```


如果我们开启了 API Aggregation，还需要为 aggregator-proxy 创建证书请求，并生成证书。
```bash
root@master:~/certs# vim aggregator-proxy-csr.json
root@master:~/certs# cat aggregator-proxy-csr.json
{
    "CN": "aggregator",
    "hosts": [],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "JS",
            "L": "SZ",
            "O": "k8s",
            "OU": "System"
        }
    ]
}

# 生成 aggregator-proxy 证书
root@master:~/certs# cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes aggregator-proxy-csr.json | cfssljson -bare aggregator-proxy
2023/05/05 10:55:04 [INFO] generate received request
2023/05/05 10:55:04 [INFO] received CSR
2023/05/05 10:55:04 [INFO] generating key: rsa-2048
2023/05/05 10:55:04 [INFO] encoded CSR
2023/05/05 10:55:04 [INFO] signed certificate with serial number 496375009815224735383401228907062774376773442957
2023/05/05 10:55:04 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
websites. For more information see the Baseline Requirements for the Issuance and Management
of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
specifically, section 10.2.3 ("Information Requirements").
root@master:~/certs# ls -l aggregator-proxy*
-rw-r--r-- 1 root root  989 May  5 10:55 aggregator-proxy.csr
-rw-r--r-- 1 root root  271 May  5 10:54 aggregator-proxy-csr.json
-rw------- 1 root root 1679 May  5 10:55 aggregator-proxy-key.pem
-rw-r--r-- 1 root root 1354 May  5 10:55 aggregator-proxy.pem
```

复制证书到证书目录
```bash
root@master:~/certs# cp {ca.pem,ca-key.pem,server.pem,server-key.pem,etcd.pem,etcd-key.pem,aggregator-proxy.pem,aggregator-proxy-key.pem} /etc/kubernetes/ssl/
```

创建 kube-apiservice 文件
```bash
root@master:~/certs# vim /usr/lib/systemd/system/kube-apiserver.service
root@master:~/certs# cat /usr/lib/systemd/system/kube-apiserver.service
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=network.target

[Service]
ExecStart=/usr/local/bin/kube-apiserver \
  --allow-privileged=true \
  --anonymous-auth=false \
  --api-audiences=api,istio-ca \
  --authorization-mode=Node,RBAC \
  --bind-address=10.0.0.99 \
  --client-ca-file=/etc/kubernetes/ssl/ca.pem \
  --enable-bootstrap-token-auth=true \
  --etcd-cafile=/etc/kubernetes/ssl/ca.pem \
  --etcd-certfile=/etc/kubernetes/ssl/etcd.pem \
  --etcd-keyfile=/etc/kubernetes/ssl/etcd-key.pem \
  --etcd-servers=https://10.0.0.99:2379,https://10.0.0.176:2379,https://10.0.2.169:2379 \
  --kubelet-client-certificate=/etc/kubernetes/ssl/server.pem \
  --kubelet-client-key=/etc/kubernetes/ssl/server-key.pem \
  --proxy-client-cert-file=/etc/kubernetes/ssl/aggregator-proxy.pem \
  --proxy-client-key-file=/etc/kubernetes/ssl/aggregator-proxy-key.pem \
  --requestheader-allowed-names= \
  --requestheader-client-ca-file=/etc/kubernetes/ssl/ca.pem \
  --requestheader-extra-headers-prefix=X-Remote-Extra- \
  --requestheader-group-headers=X-Remote-Group \
  --requestheader-username-headers=X-Remote-User \
  --enable-aggregator-routing=true \
  --secure-port=6443 \
  --service-account-issuer=https://kubernetes.default.svc \
  --service-account-key-file=/etc/kubernetes/ssl/ca.pem \
  --service-account-signing-key-file=/etc/kubernetes/ssl/ca-key.pem \
  --service-cluster-ip-range=192.168.0.0/16 \
  --service-node-port-range=30000-32767 \
  --tls-cert-file=/etc/kubernetes/ssl/server.pem \
  --tls-private-key-file=/etc/kubernetes/ssl/server-key.pem \
  --enable-aggregator-routing=true \
  --v=2
Restart=always
RestartSec=5
Type=notify
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```
--allow-privileged：是否允许特权容器，默认是 false  
--anonymous-auth：是否允许匿名请求，默认是 true  
--bind-address：用来监听 --secure-port 端口的 IP 地址。如果未指定，或指定为 0.0.0.0 则表示监听所有地址  
--client-ca-file：CA 证书  
--etcd-cafile=：etcd 使用的 root CA 证书（证书颁发机构证书）  
--etcd-certfile：etcd 的客户端证书  
--etcd-keyfile=：etcd 的客户端证书的私钥  
--etcd-servers：etcd 服务器的 URL 列表  
--secure-port：https 端口，默认 6443  
--service-cluster-ip-range：service IP 的地址范围  
--service-node-port-range：Service 可以使用的端口范围  
--tls-cert-file：API 服务端 CA 证书  
--tls-private-key-file：API 服务端 CA 私钥

对于初学者，如果你不知道应该使用那些参数，可以通过 kubeadm 先安装一个测试环境，然后通过相应组件的 yaml 文件内的启动参数，作为你启动参数，然后再根据实际情环境做一些调整。

更多参数相关书名，请参考 [kube-apiserver](https://kubernetes.io/zh-cn/docs/reference/command-line-tools-reference/kube-apiserver)


### 配置 kube-controller-manager
创建 kube-controller-manager 证书请求
```bash
root@master:~/certs# vim kube-controller-manager-csr.json
root@master:~/certs# cat kube-controller-manager-csr.json
{
    "CN": "system:kube-controller-manager",
    "hosts": [],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "JS",
            "L": "SZ",
            "O": "system:kube-controller-manager",
            "OU": "System"
        }
    ]
}

生成 kube-controller-manager 证书
root@master:~/certs# cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-controller-manager-csr.json |cfssljson -bare kube-controller-manager
2023/05/05 11:06:27 [INFO] generate received request
2023/05/05 11:06:27 [INFO] received CSR
2023/05/05 11:06:27 [INFO] generating key: rsa-2048
2023/05/05 11:06:27 [INFO] encoded CSR
2023/05/05 11:06:27 [INFO] signed certificate with serial number 663963107500902095221274457981748810134286505215
2023/05/05 11:06:27 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
websites. For more information see the Baseline Requirements for the Issuance and Management
of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
specifically, section 10.2.3 ("Information Requirements").
root@master:~/certs# ls -l kube-controller-manager*
-rw-r--r-- 1 root root 1054 May  5 11:06 kube-controller-manager.csr
-rw-r--r-- 1 root root  318 May  5 11:06 kube-controller-manager-csr.json
-rw------- 1 root root 1679 May  5 11:06 kube-controller-manager-key.pem
-rw-r--r-- 1 root root 1419 May  5 11:06 kube-controller-manager.pem
```

生成 kubeconfig 文件
```bash
root@master:~/certs# kubectl config set-cluster kubernetes --certificate-authority=ca.pem --embed-certs=true --server=https://10.0.0.99:6443 --kubeconfig=kube-controller-manager.kubeconfig
Cluster "kubernetes" set.
root@master:~/certs# kubectl config set-credentials system:kube-controller-manager --client-certificate=kube-controller-manager.pem --client-key=kube-controller-manager-key.pem --embed-certs=true --kubeconfig=kube-controller-manager.kubeconfig
User "system:kube-controller-manager" set.
root@master:~/certs# kubectl config set-context default --cluster=kubernetes --user=system:kube-controller-manager --kubeconfig=kube-controller-manager.kubeconfig
Context "default" created.
root@master:~/certs# kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig
Switched to context "default".
```
--server 用于指定 kube-apiserver 地址

复制到 /etc/kubernetes 目录
```bash
root@master:~/certs# cp kube-controller-manager.kubeconfig /etc/kubernetes/
```

创建 kube-controller service 文件
```bash
root@master:~/certs# vim /usr/lib/systemd/system/kube-controller-manager.service
root@master:~/certs# cat /usr/lib/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/GoogleCloudPlatform/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \
  --allocate-node-cidrs=true \
  --authentication-kubeconfig=/etc/kubernetes/kube-controller-manager.kubeconfig \
  --authorization-kubeconfig=/etc/kubernetes/kube-controller-manager.kubeconfig \
  --bind-address=0.0.0.0 \
  --cluster-cidr=172.16.0.0/16 \
  --cluster-name=kubernetes \
  --cluster-signing-cert-file=/etc/kubernetes/ssl/ca.pem \
  --cluster-signing-key-file=/etc/kubernetes/ssl/ca-key.pem \
  --kubeconfig=/etc/kubernetes/kube-controller-manager.kubeconfig \
  --leader-elect=true \
  --node-cidr-mask-size=24 \
  --root-ca-file=/etc/kubernetes/ssl/ca.pem \
  --service-account-private-key-file=/etc/kubernetes/ssl/ca-key.pem \
  --service-cluster-ip-range=192.168.0.0/16 \
  --use-service-account-credentials=true \
  --v=2
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```
--cluster-cidr：Pod 的 IP 地址范围，需要和网络组件（如，calico）的一致  
--leader-elect：多节点时，使用选举的方式选择主节点  
--node-cidr-mask-size：每个节点的网络大小（Pod IP）  
--service-cluster-ip-range： service 的 IP 地址范围

完整参数列表请参考 [kube-controller-manager](https://kubernetes.io/zh-cn/docs/reference/command-line-tools-reference/kube-controller-manager) 

### 配置 kube-scheduler
创建 kube-scheduler 证书请求
```bash
root@master:~/certs# vim kube-scheduler-csr.json
root@master:~/certs# cat kube-scheduler-csr.json
{
    "CN": "system:kube-scheduler",
    "hosts": [],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "JS",
            "L": "SZ",
            "O": "system:kube-scheduler",
            "OU": "System"
        }
    ]
}
```

生成 kube-scheduler 证书
```bash
root@master:~/certs# cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-scheduler-csr.json | cfssljson -bare kube-scheduler
2023/05/05 11:18:26 [INFO] generate received request
2023/05/05 11:18:26 [INFO] received CSR
2023/05/05 11:18:26 [INFO] generating key: rsa-2048
2023/05/05 11:18:26 [INFO] encoded CSR
2023/05/05 11:18:26 [INFO] signed certificate with serial number 711043720452054011592744422560522408521743614713
2023/05/05 11:18:26 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
websites. For more information see the Baseline Requirements for the Issuance and Management
of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
specifically, section 10.2.3 ("Information Requirements").
root@master:~/certs# ls -l kube-scheduler*
-rw-r--r-- 1 root root 1025 May  5 11:18 kube-scheduler.csr
-rw-r--r-- 1 root root  300 May  5 11:17 kube-scheduler-csr.json
-rw------- 1 root root 1675 May  5 11:18 kube-scheduler-key.pem
-rw-r--r-- 1 root root 1395 May  5 11:18 kube-scheduler.pem
```

生成 kubeconfig 文件
```bash
root@master:~/certs# kubectl config set-cluster kubernetes --certificate-authority=ca.pem --embed-certs=true --server=https://10.0.0.99:6443 --kubeconfig=kube-scheduler.kubeconfig
Cluster "kubernetes" set.
root@master:~/certs# kubectl config set-credentials system:kube-scheduler --client-certificate=kube-scheduler.pem --client-key=kube-scheduler-key.pem --embed-certs=true --kubeconfig=kube-scheduler.kubeconfig
User "system:kube-scheduler" set.
root@master:~/certs# kubectl config set-context default --cluster=kubernetes --user=system:kube-scheduler --kubeconfig=kube-scheduler.kubeconfig
Context "default" created.
root@master:~/certs# kubectl config use-context default --kubeconfig=kube-scheduler.kubeconfig
Switched to context "default".
```
--server 用于指定 kube-apiserver 地址


复制 kubeconfig  文件到 /etc/kubernetes
```bash
root@master:~/certs# cp kube-scheduler.kubeconfig /etc/kubernetes/
```

创建 kube-scheduler service 文件
```bash
root@master:~/certs# vim /usr/lib/systemd/system/kube-scheduler.service
root@master:~/certs# cat /usr/lib/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/GoogleCloudPlatform/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \
  --authentication-kubeconfig=/etc/kubernetes/kube-scheduler.kubeconfig \
  --authorization-kubeconfig=/etc/kubernetes/kube-scheduler.kubeconfig \
  --bind-address=0.0.0.0 \
  --kubeconfig=/etc/kubernetes/kube-scheduler.kubeconfig \
  --leader-elect=true \
  --v=2
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

完整参数列表请参考 [kube-scheduler](https://kubernetes.io/zh-cn/docs/reference/command-line-tools-reference/kube-scheduler)


启动所有组件服务
```bash
root@master:~/certs# systemctl daemon-reload

root@master:~/certs# systemctl enable --now kube-apiserver.service 
Created symlink /etc/systemd/system/multi-user.target.wants/kube-apiserver.service → /lib/systemd/system/kube-apiserver.service.

root@master:~/certs# systemctl enable --now kube-controller-manager.service 
Created symlink /etc/systemd/system/multi-user.target.wants/kube-controller-manager.service → /lib/systemd/system/kube-controller-manager.service.

root@master:~/certs# systemctl enable --now kube-scheduler.service 
Created symlink /etc/systemd/system/multi-user.target.wants/kube-scheduler.service → /lib/systemd/system/kube-scheduler.service.
```

验证 Master 节点
```bash
root@master:~/certs# kubectl get componentstatus
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS    MESSAGE                         ERROR
scheduler            Healthy   ok                              
controller-manager   Healthy   ok                              
etcd-2               Healthy   {"health":"true","reason":""}   
etcd-0               Healthy   {"health":"true","reason":""}   
etcd-1               Healthy   {"health":"true","reason":""}
```