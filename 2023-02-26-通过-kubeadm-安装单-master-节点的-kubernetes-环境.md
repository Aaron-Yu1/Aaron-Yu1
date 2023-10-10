---
title: 通过 Kubeadm 安装单 Master 节点的 kubernetes 环境
thumbnailImagePosition: left
thumbnailImage: https://img.freepik.com/premium-vector/wheel-helm-computer-developer-app-concept-business-digital-open-source-program-data-coding-steering_115739-1863.jpg
date: 2023-02-26
categories:
- Kubernetes
tags:
- Kubernetes
- CI/CD
---

今天我们来看一看，如何通过 kubeadm 来安装 kubernetes。

<!--more-->

准备三台主机（虚拟机），并配置好主机名和 IP
- Master：10.0.0.99
- Node1：10.0.0.176
- Node2: 10.0.2.169

操作系统版本：
```bash
root@master:~# lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 20.04.2 LTS
Release:    20.04
Codename:   focal
```

Container Runtime
- Containerd

### 初始化系统 (在所有节点执行)

重命名
```bash
root@eiqwjjjppqbn4niz:~# hostnamectl set-hostname master.test.local
root@eiqwjjjppqbn4niz-0001:~# hostnamectl set-hostname node1.test.local
root@eiqwjjjppqbn4niz-0002:~# hostnamectl set-hostname node2.test.local
```

```bash
# 更改 /etc/hosts 文件（添加）
root@master:~# vim /etc/hosts
root@master:~# cat /etc/hosts | tail -n 3
10.0.0.99   master.test.local
10.0.0.176  node1.test.local
10.0.2.169  node2.test.lcoal

# 更改内核参数
root@master:~# tee /etc/modules-load.d/containerd.conf  <<EOF
overlay
br_netfilter
EOF
root@master:~# modprobe overlay
root@master:~# modprobe br_netfilter

root@master:~# tee /etc/sysctl.d/k8s.conf  <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
root@master:~# sysctl -p /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1

# IPVS module
root@master:~# mkdir -p /etc/sysconfig/modules
root@master:~# vim /etc/sysconfig/modules/ipvs.modules
root@master:~# cat /etc/sysconfig/modules/ipvs.modules
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack
root@master:~# chmod 755 /etc/sysconfig/modules/ipvs.modules
root@master:~# bash /etc/sysconfig/modules/ipvs.modules
root@master:~# lsmod | grep -e ip_vs -e nf_conntrack
ip_vs_sh               16384  0
ip_vs_wrr              16384  0
ip_vs_rr               16384  0
ip_vs                 155648  6 ip_vs_rr,ip_vs_sh,ip_vs_wrr
nf_conntrack          139264  1 ip_vs
nf_defrag_ipv6         24576  2 nf_conntrack,ip_vs
nf_defrag_ipv4         16384  1 nf_conntrack
libcrc32c              16384  4 nf_conntrack,btrfs,raid456,ip_vs

# 关闭 Swap
root@master:~# sed -i 's/.*swap.*/#&/' /etc/fstab
root@master:~# cat /etc/fstab | tail -n2
/dev/disk/by-uuid/2a95978d-b614-45a5-9246-8a4b11300562 / ext4 defaults 0 0
#/swap.img  none    swap    sw  0   0
root@master:~# swapoff -a
root@master:~# free 
              total        used        free      shared  buff/cache   available
Mem:        4030540      172972     3110268        1384      747300     3621280
```

### 安装 Containerd (在所有节点执行)

```bash
# 配置安装源(该方式只适合 20.04, 22.04 添加 gpg key 的方式和 20.04 不同)
root@master:~# apt-get -y install apt-transport-https ca-certificates curl software-properties-common
root@node1:~# curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
root@master:~# add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
root@master:~# apt update 

# 查看可以安装的版本，选择合适的版本，或最新版
root@master:~# apt-cache madison containerd.io
containerd.io |   1.6.18-1 | http://mirrors.aliyun.com/docker-ce/linux/ubuntu focal/stable amd64 Packages
containerd.io |   1.6.16-1 | http://mirrors.aliyun.com/docker-ce/linux/ubuntu focal/stable amd64 Packages
containerd.io |   1.6.15-1 | http://mirrors.aliyun.com/docker-ce/linux/ubuntu focal/stable amd64 Packages
containerd.io |   1.6.14-1 | http://mirrors.aliyun.com/docker-ce/linux/ubuntu focal/stable amd64 Packages
... ...
containerd.io |    1.4.3-1 | http://mirrors.aliyun.com/docker-ce/linux/ubuntu focal/stable amd64 Packages
containerd.io |    1.3.9-1 | http://mirrors.aliyun.com/docker-ce/linux/ubuntu focal/stable amd64 Packages
containerd.io |    1.3.7-1 | http://mirrors.aliyun.com/docker-ce/linux/ubuntu focal/stable amd64 Packages
containerd.io |   1.2.13-2 | http://mirrors.aliyun.com/docker-ce/linux/ubuntu focal/stable amd64 Packages

# 安装 containerd
root@master:~# apt install -y containerd.io
root@master:~# systemctl status containerd.service
● containerd.service - containerd container runtime
     Loaded: loaded (/lib/systemd/system/containerd.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2023-02-19 12:15:17 CST; 2min 53s ago
       Docs: https://containerd.io
    Process: 5389 ExecStartPre=/sbin/modprobe overlay (code=exited, status=0/SUCCESS)
   Main PID: 5391 (containerd)
      Tasks: 8
     Memory: 12.5M
     CGroup: /system.slice/containerd.service
             └─5391 /usr/bin/containerd

Feb 19 12:15:17 master.test.local containerd[5391]: time="2023-02-19T12:15:17.175037835+08:00" level=info msg="loading plugin \"io.containerd.grpc.v1.tasks\"..." type=>
Feb 19 12:15:17 master.test.local containerd[5391]: time="2023-02-19T12:15:17.175056371+08:00" level=info msg="loading plugin \"io.containerd.grpc.v1.version\"..." typ>
Feb 19 12:15:17 master.test.local containerd[5391]: time="2023-02-19T12:15:17.175077081+08:00" level=info msg="loading plugin \"io.containerd.tracing.processor.v1.otlp>
Feb 19 12:15:17 master.test.local containerd[5391]: time="2023-02-19T12:15:17.175095265+08:00" level=info msg="skip loading plugin \"io.containerd.tracing.processor.v1>
Feb 19 12:15:17 master.test.local containerd[5391]: time="2023-02-19T12:15:17.175107978+08:00" level=info msg="loading plugin \"io.containerd.internal.v1.tracing\"...">
Feb 19 12:15:17 master.test.local containerd[5391]: time="2023-02-19T12:15:17.175133781+08:00" level=error msg="failed to initialize a tracing processor \"otlp\"" erro>
Feb 19 12:15:17 master.test.local containerd[5391]: time="2023-02-19T12:15:17.175574846+08:00" level=info msg=serving... address=/run/containerd/containerd.sock.ttrpc
Feb 19 12:15:17 master.test.local containerd[5391]: time="2023-02-19T12:15:17.175647561+08:00" level=info msg=serving... address=/run/containerd/containerd.sock
Feb 19 12:15:17 master.test.local containerd[5391]: time="2023-02-19T12:15:17.175735626+08:00" level=info msg="containerd successfully booted in 0.051058s"
Feb 19 12:15:17 master.test.local systemd[1]: Started containerd container runtime.

# 生成配置文件，更改 sandbox 镜像地址到国内地址
root@master:~# containerd config default | tee /etc/containerd/config.toml
root@master:~# vim /etc/containerd/config.toml
root@master:~# cat /etc/containerd/config.toml | grep -e sandbox -e System
    sandbox_image = "registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.6"
            SystemdCgroup = true           # 注意 S 和 C 都是大写
root@master:~# systemctl restart containerd.service
```

### 安装 Kubernets 组件 (在所有节点执行)
```bash
# 配置使用国内源
root@master:~# curl -fsSL https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -
OK
root@master:~# cat &lt;&lt;EOF &gt;/etc/apt/sources.list.d/kubernetes.list
> deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
> EOF
root@master:~# apt update 

# 查看可用的版本，选用合适的版本，或最新版
root@master:~# apt-cache madison kubeadm 
   kubeadm |  1.26.1-00 | https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial/main amd64 Packages
   kubeadm |  1.26.0-00 | https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial/main amd64 Packages
   kubeadm |  1.25.6-00 | https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial/main amd64 Packages
   kubeadm |  1.25.5-00 | https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial/main amd64 Packages
   kubeadm |  1.25.4-00 | https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial/main amd64 Packages
   kubeadm |  1.25.3-00 | https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial/main amd64 Packages
   ... ...
   kubeadm | 1.22.12-00 | https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial/main amd64 Packages
   kubeadm | 1.22.11-00 | https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial/main amd64 Packages
   kubeadm | 1.22.10-00 | https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial/main amd64 Packages
... ...
... ...

# 安装 Kubernetes 组件，并指定版本(Node 节点可以不安装 kubectl)
root@master:~# apt install -y kubeadm=1.25.0-00 kubelet=1.25.0-00 kubectl=1.25.0-00

# 更改 service 文件，在 ExecStart=/usr/bin/kubelet 后面添加 container 信息
root@master:~# vim /usr/lib/systemd/system/kubelet.service
root@master:~# cat /usr/lib/systemd/system/kubelet.service
[Unit]
Description=kubelet: The Kubernetes Node Agent
Documentation=https://kubernetes.io/docs/home/
Wants=network-online.target
After=network-online.target

[Service]
ExecStart=/usr/bin/kubelet --container-runtime=remote --runtime-request-timeout=15m --container-runtime-endpoint=unix:///run/containerd/containerd.sock
Restart=always
StartLimitInterval=0
RestartSec=10

[Install]
WantedBy=multi-user.target

# 启动服务，并设置开机启动
root@master:~# systemctl daemon-reload 
it@master:~$ sudo systemctl enable --now kubelet
```

### 初始化 Master 节点（只需要在 Master 节点上执行）
```bash
# 生成默认地配置文件
root@master:~# kubeadm config print init-defaults  &gt; kubeadm-config.yaml

# 修改配置文件
root@master:~# vim kubeadm-config.yaml 
root@master:~# cat kubeadm-config.yaml
apiVersion: kubeadm.k8s.io/v1beta3
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 10.0.0.99        # API Server 节点 IP 地址
  bindPort: 6443
nodeRegistration:
  criSocket: unix:///var/run/containerd/containerd.sock
  imagePullPolicy: IfNotPresent
  name: node
  taints: null
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta3
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns: {}
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers        # 镜像仓库地址改成国内地址
kind: ClusterConfiguration
kubernetesVersion: 1.25.0
networking:
  dnsDomain: test.local               # 域名
  podSubnet: 172.16.0.0/16            # Pod 子网
  serviceSubnet: 10.96.0.0/12         # Service 子网
scheduler: {}

# 使用该配置文件初始化 Master 节点
root@master:~# kubeadm init --config kubeadm-config.yaml
[init] Using Kubernetes version: v1.25.0
[preflight] Running pre-flight checks
    [WARNING Hostname]: hostname "node" could not be reached
    [WARNING Hostname]: hostname "node": lookup node on 114.114.114.114:53: no such host
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using &#039;kubeadm config images pull&#039;
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
... ...
... ...
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!      # 初始化已经完成

To start using your cluster, you need to run the following as a regular user:      # 初始化完成后，还需要执行下面的步骤

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.                   # 初始化完成后还需要部署网络插件
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/              # 该站点列出了支持的的网络插件

Then you can join any number of worker nodes by running the following on each as root:     # 在 node 运行下面命令，将 Node 节点加入到群集

kubeadm join 10.0.0.99:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:3439058d679bb0683ad06a08afd8936aa9bcf8616aa73ce1e89fc44868ff39ac 

# 执行初始化之后的操作
# 复制 admin config 文件
root@master:~# mkdir -p $HOME/.kube
root@master:~# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
root@master:~# sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 查看节点状态（没有网络插件，所以 status 是 NotReady
root@master:~# kubectl get nodes
NAME   STATUS     ROLES           AGE    VERSION
node   NotReady   control-plane   4m3s   v1.25.0
```

### 部署网络插件（以 calico 为例, 也是只需要在 Master 节点上执行）

查看指定版本的 [requirements](https://docs.tigera.io/calico/3.25/getting-started/kubernetes/requirements) 页面，以确定 calico 的版本是否支持当前的 kubernetes 版本.  


以 3.25 为例
```bash
    Kubernetes requirements
    Supported versions
    We test Calico v3.25 against the following Kubernetes versions.
    
    v1.23
    v1.24
    v1.25
    v1.26
```


有两种方式安装
- Operator
- Manifest

更多信息，请参考 [Install Calico](https://docs.tigera.io/calico/3.25/getting-started/kubernetes/self-managed-onprem/onpremises#install-calico-with-etcd-datastore)。

我们这里以 Manifest 方式为例
```bash
root@master:~# wget https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml

# 更改地址池范围，使其与 Kubernetes 配置文件中的 podSubnet 一致
root@master:~# vim calico.yaml 
root@master:~# cat calico.yaml | grep CALICO_IPV4POOL_CIDR -A 1
            - name: CALICO_IPV4POOL_CIDR
              value: "172.16.0.0/16"

# 安装网络插件
root@master:~# kubectl apply -f calico.yaml

# 再次验证节点状态，此时变成了 Ready
root@master:~# kubectl get nodes
NAME   STATUS   ROLES           AGE   VERSION
node   Ready    control-plane   31m   v1.25.0
root@master:~# kubectl get pods -A         # calico 以 pod 的形式在节点上运行
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-74677b4c5f-prfpp   1/1     Running   0          80s
kube-system   calico-node-5xv2s                          1/1     Running   0          80s
kube-system   coredns-7f8cbcb969-6mpk8                   1/1     Running   0          31m
kube-system   coredns-7f8cbcb969-mxwpz                   1/1     Running   0          31m
kube-system   etcd-node                                  1/1     Running   0          31m
kube-system   kube-apiserver-node                        1/1     Running   0          31m
kube-system   kube-controller-manager-node               1/1     Running   0          31m
kube-system   kube-proxy-mp5t7                           1/1     Running   0          31m
kube-system   kube-scheduler-node                        1/1     Running   0          31m
```

### 添加 Node 节点到群集（只需要在 work 节点上执行）
```bash
root@node1:~# kubeadm join 10.0.0.99:6443 --token abcdef.0123456789abcdef \
> --discovery-token-ca-cert-hash sha256:3439058d679bb0683ad06a08afd8936aa9bcf8616aa73ce1e89fc44868ff39ac 
... ...
... ...
This node has joined the cluster:            # 节点被添加到群集
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run &#039;kubectl get nodes&#039; on the control-plane to see this node join the cluster.      # 在管理平面（Master 节点）运行命令查看节点状态

# 返回 Master 节点查看节点状态（Ready）
root@master:~# kubectl get nodes
NAME               STATUS   ROLES           AGE   VERSION
node               Ready    control-plane   34m   v1.25.0
node1.test.local   Ready    <none>          53s   v1.25.0
node2.test.local   Ready    <none>          49s   v1.25.0
root@master:~# kubectl get pods -A           # 增加了两个新的 calico pod
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-74677b4c5f-prfpp   1/1     Running   0          4m19s
kube-system   calico-node-5xv2s                          1/1     Running   0          4m19s
kube-system   calico-node-bxzxl                          1/1     Running   0          72s
kube-system   calico-node-gxwk5                          1/1     Running   0          76s
kube-system   coredns-7f8cbcb969-6mpk8                   1/1     Running   0          34m
kube-system   coredns-7f8cbcb969-mxwpz                   1/1     Running   0          34m
kube-system   etcd-node                                  1/1     Running   0          34m
kube-system   kube-apiserver-node                        1/1     Running   0          34m
kube-system   kube-controller-manager-node               1/1     Running   0          34m
kube-system   kube-proxy-hnqbg                           1/1     Running   0          72s
kube-system   kube-proxy-mp5t7                           1/1     Running   0          34m
kube-system   kube-proxy-t6dtt                           1/1     Running   0          76s
kube-system   kube-scheduler-node                        1/1     Running   0          34m
```