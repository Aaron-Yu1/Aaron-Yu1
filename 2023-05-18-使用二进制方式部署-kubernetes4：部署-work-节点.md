---
title: 使用二进制方式部署 kubernetes(4)：部署 Work 节点
thumbnailImagePosition: left
thumbnailImage: https://img.freepik.com/premium-photo/businessman-team-analyzing-financial-statement-finance-task-with-smart-phone-laptop-tablet-wealth-management-concept_265022-8758.jpg
date: 2023-05-18
categories:
- Kubernetes
tags:
- containerd
- kubernetes
- work nodes
---

通过前面的内容，我们知道了如何部署 master 节点，现在，我们来看一看，如何部署 work 节点。

<!--more-->

### 创建 kubelet 证书

因为只有 master 节点上有 cfssl，所以整个生成证书部分的操作都是在 master 节点上完成的。

创建节点的 kubelet 证书请求
```bash
# 我们需要在 CN 中添加 node 后面添加 “:” + 节点名
# 为不同的节点生成不同的证书
root@master:~/certs# vim work1-kubelet-csr.json
root@master:~/certs# cat work1-kubelet-csr.json
{
    "CN": "system:node:work1",
    "hosts": [
        "127.0.0.1",
        "10.0.0.176",
        "work1"
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
            "O": "system:nodes",
            "OU": "System"
        }
    ]
}
root@master:~/certs# vim work2-kubelet-csr.json
root@master:~/certs# cat work2-kubelet-csr.json
{
    "CN": "system:node:work2",
    "hosts": [
        "127.0.0.1",
        "10.0.2.169",
        "work2"
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
            "O": "system:nodes",
            "OU": "System"
        }
    ]
}
```

生成节点的 kubelet 证书
```bash
root@master:~/certs# cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes work1-kubelet-csr.json | cfssljson -bare work1-kubelet
2023/05/05 11:33:22 [INFO] generate received request
2023/05/05 11:33:22 [INFO] received CSR
2023/05/05 11:33:22 [INFO] generating key: rsa-2048
2023/05/05 11:33:23 [INFO] encoded CSR
2023/05/05 11:33:23 [INFO] signed certificate with serial number 645329070658903044437744424038429758047541583282
root@master:~/certs# cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes work2-kubelet-csr.json | cfssljson -bare work2-kubelet
2023/05/05 11:33:45 [INFO] generate received request
2023/05/05 11:33:45 [INFO] received CSR
2023/05/05 11:33:45 [INFO] generating key: rsa-2048
2023/05/05 11:33:45 [INFO] encoded CSR
2023/05/05 11:33:45 [INFO] signed certificate with serial number 171495683478368987085954170536850984909986423292
root@master:~/certs# ls -l work*
-rw-r--r-- 1 root root 1074 May  5 11:33 work1-kubelet.csr
-rw-r--r-- 1 root root  344 May  5 11:32 work1-kubelet-csr.json
-rw------- 1 root root 1675 May  5 11:33 work1-kubelet-key.pem
-rw-r--r-- 1 root root 1419 May  5 11:33 work1-kubelet.pem
-rw-r--r-- 1 root root 1074 May  5 11:33 work2-kubelet.csr
-rw-r--r-- 1 root root  344 May  5 11:31 work2-kubelet-csr.json
-rw------- 1 root root 1679 May  5 11:33 work2-kubelet-key.pem
-rw-r--r-- 1 root root 1419 May  5 11:33 work2-kubelet.pem
```


### 创建 kubelet.kubeconfig 文件

生成 work1 节点的 kubelet.kubeconfig 文件
```bash
root@master:~/certs# kubectl config set-cluster kubernetes --certificate-authority=ca.pem --embed-certs=true --server=https://10.0.0.99:6443 --kubeconfig=work1-kubelet.kubeconfig
Cluster "kubernetes" set.
root@master:~/certs# kubectl config set-credentials system:node:work1 --client-certificate=work1-kubelet.pem --embed-certs=true --client-key=work1-kubelet-key.pem --kubeconfig=work1-kubelet.kubeconfig
User "system:node:work1" set.
root@master:~/certs# kubectl config set-context default --cluster=kubernetes --user=system:node:work1 --kubeconfig=work1-kubelet.kubeconfig
Context "default" created.
root@master:~/certs# kubectl config use-context default --kubeconfig=work1-kubelet.kubeconfig
Switched to context "default".
```

生成 work2 节点的 kubelet.kubeconfig 文件
```bash
root@master:~/certs# kubectl config set-cluster kubernetes --certificate-authority=ca.pem --embed-certs=true --server=https://10.0.0.99:6443 --kubeconfig=work2-kubelet.kubeconfig
Cluster "kubernetes" set.
root@master:~/certs# kubectl config set-credentials system:node:work2 --client-certificate=work2-kubelet.pem --embed-certs=true --client-key=work2-kubelet-key.pem --kubeconfig=work2-kubelet.kubeconfig
User "system:node:work2" set.
root@master:~/certs# kubectl config set-context default --cluster=kubernetes --user=system:node:work2 --kubeconfig=work2-kubelet.kubeconfig
Context "default" created.
root@master:n/certs# kubectl config use-context default --kubeconfig=work2-kubelet.kubeconfig
Switched to context "default".
```
    
### 创建 kube-proxy 证书
创建 kube-proxy 证书请求文件
```bash
root@master:~/certs# vim kube-proxy-csr.json
root@master:~/certs# cat kube-proxy-csr.json
{
    "CN": "system:kube-proxy",
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
            "O": "system:kube-proxy",
            "OU": "System"
        }
    ]
}
```
    
生成 kube-proxy 证书
```bash
root@master:~/certs# cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-proxy-csr.json | cfssljson -bare kube-proxy
2023/05/05 16:02:01 [INFO] generate received request
2023/05/05 16:02:01 [INFO] received CSR
2023/05/05 16:02:01 [INFO] generating key: rsa-2048
2023/05/05 16:02:01 [INFO] encoded CSR
2023/05/05 16:02:01 [INFO] signed certificate with serial number 324352288328217470755205596576203430057773918495
2023/05/05 16:02:01 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
websites. For more information see the Baseline Requirements for the Issuance and Management
of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
specifically, section 10.2.3 ("Information Requirements").
root@master:~/certs# ls -l kube-proxy*
-rw-r--r-- 1 root root 1017 May  5 16:02 kube-proxy.csr
-rw-r--r-- 1 root root  292 May  5 16:01 kube-proxy-csr.json
-rw------- 1 root root 1675 May  5 16:02 kube-proxy-key.pem
-rw-r--r-- 1 root root 1383 May  5 16:02 kube-proxy.pem
```
    
### 创建 kube-proxy.kubeconfig 文件
```bash
root@master:~/certs# kubectl config set-cluster kubernetes --certificate-authority=ca.pem --embed-certs=true --server=https://10.0.0.99:6443 --kubeconfig=kube-proxy.kubeconfig
Cluster "kubernetes" set.
root@master:~/certs# kubectl config set-credentials kube-proxy --client-certificate=kube-proxy.pem --embed-certs=true --client-key=kube-proxy-key.pem --kubeconfig=kube-proxy.kubeconfig
User "kube-proxy" set.
root@master:~/certs# kubectl config set-context default --cluster=kubernetes --user=kube-proxy --kubeconfig=kube-proxy.kubeconfig
Context "default" created.
root@master:~/certs# kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
Switched to context "default".
```
### 复制文件到 work 节点

复制证书文件到 work 节点
```bash
root@master:~/certs# scp {ca.pem,work1-kubelet.pem,work1-kubelet-key.pem} work1:/etc/kubernetes/ssl
The authenticity of host 'work1 (10.0.0.176)' can't be established.
ECDSA key fingerprint is SHA256:V9HHbSQhryGjoirPPz/CyvERgb99vA+1rMseOC6N5b0.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'work1' (ECDSA) to the list of known hosts.
root@work1's password: 
ca.pem                                                                           100% 1261     1.7MB/s   00:00    
work1-kubelet.pem                                                                100% 1419     2.8MB/s   00:00    
work1-kubelet-key.pem                                                            100% 1675     3.6MB/s   00:00 

root@master:~/certs# scp {ca.pem,work2-kubelet.pem,work2-kubelet-key.pem} work2:/etc/kubernetes/ssl
The authenticity of host 'work2 (10.0.2.169)' can't be established.
ECDSA key fingerprint is SHA256:MGH1ZyPJkHB9wFCt3SbNGbSU5YmV4n2wDFXaYS4y0GY.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'work2' (ECDSA) to the list of known hosts.
root@work2's password: 
ca.pem                                                                           100% 1261     2.3MB/s   00:00    
work2-kubelet.pem                                                                100% 1419     1.0MB/s   00:00    
work2-kubelet-key.pem                                                            100% 1679     3.4MB/s   00:00
```

复制 kubelet.kubeconfig 文件到 work 节点
```bash
root@master:~/certs# scp work1-kubelet.kubeconfig work1:/etc/kubernetes/kubelet.kubeconfig
root@work1's password: 
work1-kubelet.kubeconfig                                                         100% 6171     6.8MB/s   00:00    
root@master:~/certs# scp work2-kubelet.kubeconfig work2:/etc/kubernetes/kubelet.kubeconfig
root@work2's password: 
work2-kubelet.kubeconfig                                                         100% 6175     8.3MB/s   00:00  
```

复制 kube-proxy.kubeconfig 到 work 节点
```bash
root@master:~/certs# scp kube-proxy.kubeconfig work1:/etc/kubernetes/
root@work1's password: 
kube-proxy.kubeconfig                                                            100% 6109     9.3MB/s   00:00    
root@master:~/certs# scp kube-proxy.kubeconfig work2:/etc/kubernetes/
root@work2's password: 
kube-proxy.kubeconfig                                                            100% 6109     6.2MB/s   00:00    
```

复制二进制文件到 work 节点
```bash
root@master:~/certs# scp ../pkg/kubernetes/server/bin/{kubelet,kube-proxy} work1:/usr/local/bin/
root@work1's password: 
kubelet                                                                          100%  116MB  85.4MB/s   00:01    
kube-proxy                                                                       100%   43MB  62.1MB/s   00:00    
root@master:~/certs# scp ../pkg/kubernetes/server/bin/{kubelet,kube-proxy} work2:/usr/local/bin/
root@work2's password: 
kubelet                                                                          100%  116MB  79.3MB/s   00:01    
kube-proxy                                                                       100%   43MB  73.3MB/s   00:00
```


### 安装 Containerd（在所有 work 节点）
```bash
root@work1:~# wget http://download.opscoffee.com/cri-containerd-cni-1.6.19-linux-amd64.tar.gz
root@work1:~# tar -zxf cri-containerd-cni-1.6.19-linux-amd64.tar.gz 

root@work1:~# cp usr/local/bin/* /usr/local/bin/
root@work1:~# scp usr/local/bin/* work2:/usr/local/bin/

root@work1:~# cp usr/local/sbin/* /usr/local/sbin/
root@work1:~# scp usr/local/sbin/* work2:/usr/local/sbin/

root@work1:~# cp -r opt/cni /opt/
root@work1:~# scp -r opt/cni work2:/opt/

root@work1:~# cp -r opt/containerd /opt/
root@work1:~# scp -r opt/containerd work2:/opt/

root@work1:~# cp -r etc/cni /etc/
root@work1:~# scp -r etc/cni work2:/etc/

root@work1:~# cp etc/crictl.yaml /etc/
root@work1:~# scp etc/crictl.yaml work2:/etc/

root@work1:~# cp etc/systemd/system/containerd.service /etc/systemd/system/
root@work1:~# scp etc/systemd/system/containerd.service work2:/etc/systemd/system/

# 生成默认的 containerd 配置文件
root@work1:~# mkdir /etc/containerd/
root@work1:~# containerd config default > /etc/containerd/config.toml

# 将默认的 pause 镜像下载地址改到国内，并将 SystemdCgroup 的值改为 true
root@work1:~# sed -i 's/    sandbox_image.*/    sandbox_image = "registry.aliyuncs.com\/google_containers\/pause:3.6"/g' /etc/containerd/config.toml
root@work1:~# sed -i 's/            SystemdCgroup = false/            SystemdCgroup = true/g' /etc/containerd/config.toml
root@work1:~# cat /etc/containerd/config.toml | grep -e sandbox -e SystemdC
    sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.6"
            SystemdCgroup = true
root@work1:~# scp -r /etc/containerd work2:/etc/

# 启动服务
root@work1:~# systemctl daemon-reload 
root@work1:~# systemctl enable --now containerd.service 
Created symlink /etc/systemd/system/multi-user.target.wants/containerd.service → /etc/systemd/system/containerd.service.

root@work1:~# systemctl status containerd.service 
● containerd.service - containerd container runtime
     Loaded: loaded (/etc/systemd/system/containerd.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2023-05-05 15:28:35 CST; 5s ago
       Docs: https://containerd.io
    Process: 227531 ExecStartPre=/sbin/modprobe overlay (code=exited, status=0/SUCCESS)
   Main PID: 227540 (containerd)
      Tasks: 10
     Memory: 13.5M
     CGroup: /system.slice/containerd.service
             └─227540 /usr/local/bin/containerd

May 05 15:28:35 work1 containerd[227540]: time="2023-05-05T15:28:35.278262718+08:00" level=info msg=serving... add>
May 05 15:28:35 work1 containerd[227540]: time="2023-05-05T15:28:35.278307995+08:00" level=info msg=serving... add>
May 05 15:28:35 work1 containerd[227540]: time="2023-05-05T15:28:35.278349419+08:00" level=info msg="containerd su>
May 05 15:28:35 work1 systemd[1]: Started containerd container runtime.
May 05 15:28:35 work1 containerd[227540]: time="2023-05-05T15:28:35.295890158+08:00" level=info msg="Start subscri.
May 05 15:28:35 work1 containerd[227540]: time="2023-05-05T15:28:35.296581043+08:00" level=info msg="Start recover>
May 05 15:28:35 work1 containerd[227540]: time="2023-05-05T15:28:35.297187918+08:00" level=info msg="Start event m>
May 05 15:28:35 work1 containerd[227540]: time="2023-05-05T15:28:35.297339134+08:00" level=info msg="Start snapsho>
May 05 15:28:35 work1 containerd[227540]: time="2023-05-05T15:28:35.297449942+08:00" level=info msg="Start cni net>
May 05 15:28:35 work1 containerd[227540]: time="2023-05-05T15:28:35.297548075+08:00" level=info msg="Start streami>
```

### 配置 kubelet
需要在所有节点执行，证书文件中的节点名需要去掉
```bash
root@work1:~# mkdir /var/lib/{kubelet,kube-proxy}
root@work1:~# mv /etc/kubernetes/ssl/work1-kubelet.pem /etc/kubernetes/ssl/kubelet.pem 
root@work1:~# mv /etc/kubernetes/ssl/work1-kubelet-key.pem /etc/kubernetes/ssl/kubelet-key.pem
```


创建 service 文件
```bash
root@work1:~# vim /usr/lib/systemd/system/kubelet.service
root@work1:~# cat /usr/lib/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/GoogleCloudPlatform/kubernetes

[Service]
WorkingDirectory=/var/lib/kubelet
ExecStart=/usr/local/bin/kubelet \
  --config=/var/lib/kubelet/config.yaml \
  --container-runtime-endpoint=unix:///run/containerd/containerd.sock \
  --hostname-override=work1 \
  --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.6 \
  --kubeconfig=/etc/kubernetes/kubelet.kubeconfig \
  --root-dir=/var/lib/kubelet \
  --v=2
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```
--config：指定 kubelet 的配置文件  
--container-runtime-endpoint： 指定 runtime  
--hostname-override： 节点名称  
--pod-infra-container-image：国内不能访问 Google 镜像仓库，所以需要指定到国内的地址来下载 pause 镜像

完整参数列表请参考 [kubelet](https://kubernetes.io/zh-cn/docs/reference/command-line-tools-reference/kubelet)

创建 kubelet 的配置文件
```bash
root@work1:~# vim /var/lib/kubelet/config.yaml
root@work1:~# cat /var/lib/kubelet/config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
address: 0.0.0.0
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 2m0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/ssl/ca.pem
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 5m0s
    cacheUnauthorizedTTL: 30s
cgroupDriver: systemd 
cgroupsPerQOS: true
clusterDNS:
- 172.16.0.2
clusterDomain: cluster.local
configMapAndSecretChangeDetectionStrategy: Watch
containerLogMaxFiles: 3 
containerLogMaxSize: 10Mi
enforceNodeAllocatable:
- pods
eventBurst: 10
eventRecordQPS: 5
evictionHard:
  imagefs.available: 15%
  memory.available: 300Mi
  nodefs.available: 10%
  nodefs.inodesFree: 5%
evictionPressureTransitionPeriod: 5m0s
failSwapOn: true
fileCheckFrequency: 40s
hairpinMode: hairpin-veth 
healthzBindAddress: 0.0.0.0
healthzPort: 10248
httpCheckFrequency: 40s
imageGCHighThresholdPercent: 85
imageGCLowThresholdPercent: 80
imageMinimumGCAge: 2m0s
kubeAPIBurst: 100
kubeAPIQPS: 50
makeIPTablesUtilChains: true
maxOpenFiles: 1000000
maxPods: 110
nodeLeaseDurationSeconds: 40
nodeStatusReportFrequency: 1m0s
nodeStatusUpdateFrequency: 10s
oomScoreAdj: -999
podPidsLimit: -1
port: 10250
# disable readOnlyPort 
readOnlyPort: 0
resolvConf: /run/systemd/resolve/resolv.conf
runtimeRequestTimeout: 2m0s
serializeImagePulls: true
streamingConnectionIdleTimeout: 4h0m0s
syncFrequency: 1m0s
tlsCertFile: /etc/kubernetes/ssl/kubelet.pem
tlsPrivateKeyFile: /etc/kubernetes/ssl/kubelet-key.pem
```
clusterDNS 指定群集的 DNS，这个需要根据你实际的网络环境来定。一般情况下，网段的第一个地址是 kubernetes 自己，我们会将第二个 IP 作为 Cluster DNS 的地址。

kubelet 配置文件相关参数可以参考：
- [通过配置文件设置 kubelet 参数](https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/kubelet-config-file)
- [Kubelet 配置](https://kubernetes.io/zh-cn/docs/reference/config-api/kubelet-config.v1beta1)

复制 kubelet service 和配置文件到其他节点，并修改
```bash
root@work1:~# scp /var/lib/kubelet/config.yaml work2:/var/lib/kubelet
root@work1:~# scp /usr/lib/systemd/system/kubelet.service work2:/usr/lib/systemd/system/

# 在 work2 节点上更改 service 文件的
root@work2:~# vim /usr/lib/systemd/system/kubelet.service 
root@work2:~# cat /usr/lib/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/GoogleCloudPlatform/kubernetes

[Service]
WorkingDirectory=/var/lib/kubelet
ExecStart=/usr/local/bin/kubelet \
  --config=/var/lib/kubelet/config.yaml \
  --container-runtime-endpoint=unix:///run/containerd/containerd.sock \
  --hostname-override=work2 \
  --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.6 \
  --kubeconfig=/etc/kubernetes/kubelet.kubeconfig \
  --root-dir=/var/lib/kubelet \
  --v=2
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

### 配置 kube-proxy

创建 kube-proxy 配置文件
```bash
# mode：指定 kube-proxy 的模式（iptables 或 ipvs）
# hostnameOverride: 为节点名
root@work1:~# vim /var/lib/kube-proxy/kube-proxy-config.yaml 
root@work1:~# cat /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
bindAddress: 0.0.0.0
clientConnection:
  kubeconfig: "/etc/kubernetes/kube-proxy.kubeconfig"
clusterCIDR: "172.16.0.0/16"
conntrack:
  maxPerCore: 32768
  min: 131072
  tcpCloseWaitTimeout: 1h0m0s
  tcpEstablishedTimeout: 24h0m0s
healthzBindAddress: 0.0.0.0:10256
hostnameOverride: "work1"
metricsBindAddress: 0.0.0.0:10249
mode: "ipvs"
```

kube-proxy 配置文件参数帮助请参考 [kube-proxy 配置](https://kubernetes.io/zh-cn/docs/reference/config-api/kube-proxy-config.v1alpha1)

创建 kube-proxy service 文件
```bash
root@work1:~# vim /usr/lib/systemd/system/kube-proxy.service
root@work1:~# cat /usr/lib/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube-Proxy Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=network.target

[Service]
WorkingDirectory=/var/lib/kube-proxy
ExecStart=/usr/local/bin/kube-proxy \
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml \
  --kubeconfig=/etc/kubernetes/kube-proxy.kubeconfig
Restart=always
RestartSec=5
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

复制 kube-proxy service 和配置文件到其他节点并修改
```bash
root@work1:~# scp /var/lib/kube-proxy/kube-proxy-config.yaml work2:/var/lib/kube-proxy/
root@work1's password: 
kube-proxy-config.yaml                                                           100%  411   245.6KB/s   00:00    

root@work1:~# scp /usr/lib/systemd/system/kube-proxy.service work2:/usr/lib/systemd/system/
root@work2's password: 
kube-proxy.service                                                               100%  412   790.0KB/s   00:00 
```

更改 kube-proxy-config.yaml 文件
```bash
root@work2:~# vim /var/lib/kube-proxy/kube-proxy-config.yaml 
root@work2:~# systemctl daemon-reload 
root@work2:~# cat /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
bindAddress: 0.0.0.0
clientConnection:
  kubeconfig: "/etc/kubernetes/kube-proxy.kubeconfig"
clusterCIDR: "172.16.0.0/16"
conntrack:
  maxPerCore: 32768
  min: 131072
  tcpCloseWaitTimeout: 1h0m0s
  tcpEstablishedTimeout: 24h0m0s
healthzBindAddress: 0.0.0.0:10256
hostnameOverride: "work2"
metricsBindAddress: 0.0.0.0:10249
mode: "ipvs"
```

启动服务
```bash
root@work1:~# systemctl daemon-reload
root@work1:~# systemctl enable --now kubelet.service 
Created symlink /etc/systemd/system/multi-user.target.wants/kubelet.service → /lib/systemd/system/kubelet.service.
root@work1:~# systemctl enable --now kube-proxy.service 
Created symlink /etc/systemd/system/multi-user.target.wants/kube-proxy.service → /lib/systemd/system/kube-proxy.service.
```


回到 Master 节点验证：
```bash
root@master:~# kubectl get node
NAME    STATUS   ROLES    AGE   VERSION
work1   NotReady    <none>   12d   v1.26.4
work2   NotReady    <none>   12d   v1.26.4
```
状态是 NotReady ，先忽略它，这是因为网络插件没有安装好的原因.