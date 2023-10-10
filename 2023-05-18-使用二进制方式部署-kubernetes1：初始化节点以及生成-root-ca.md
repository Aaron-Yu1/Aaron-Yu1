---
title: 使用二进制方式部署 kubernetes(1)：初始化节点以及生成 root CA
thumbnailImagePosition: left
thumbnailImage: https://img.freepik.com/premium-photo/technology-security-concept-safety-digital-protection-system_142243-71.jpg
date: 2023-05-18
categories:
- Kubernetes
tags:
- kubernetes
- cfssl
- CA
---

之前，我们有讲过如何通过 kubeadm 来部署一个单 master 的群集。今天，我们来看一下，如何通过二进制的方式部署一个单 Master 的群集。

<!--more-->

### 配置节点主机
更改所有主机的 /etc/hosts 文件，使他们能够通过主机名通信
```bash
root@master:~# cat >> /etc/hosts <<EOF
10.0.0.99 master
10.0.0.176 work1
10.0.2.169 work2
EOF
```


关闭所有节点的 swap
```bash
root@master:~# sed -i 's/^\([^#].*swap.*\)$/#\1/g' /etc/fstab
root@master:~# swapoff -a
```

更改内核参数以启用 IP forward 等功能
```bash
root@master:~# cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
root@master:~# sysctl -p /etc/sysctl.d/k8s.conf
```

如果 kube-proxy 使用 ipvs，还需要安装 ipset，并加载指定的模块。为了便查看 ipvs 规则，还可以顺序安装 ipvsadm
```bash
root@master:~# apt install ipset ipvsadm -y
root@master:~# vim /etc/modules
root@master:~# cat /etc/modules
# /etc/modules: kernel modules to load at boot time.
#
# This file contains the names of kernel modules that should be loaded
# at boot time, one per line. Lines beginning with "#" are ignored.

br_netfilter
ip_vs
ip_vs_rr
ip_vs_wrr
ip_vs_sh
nf_conntrack
root@master:~# for i in br_netfilter ip_vs ip_vs_rr ip_vs_wrr ip_vs_sh nf_conntrack;do sudo modprobe $i;done
root@master:~# lsmod | grep -E "ip_vs|nf_conntrack"
ip_vs_sh               16384  0
ip_vs_wrr              16384  0
ip_vs_rr               16384  0
ip_vs                 155648  6 ip_vs_rr,ip_vs_sh,ip_vs_wrr
nf_conntrack          139264  1 ip_vs
nf_defrag_ipv6         24576  2 nf_conntrack,ip_vs
nf_defrag_ipv4         16384  1 nf_conntrack
libcrc32c              16384  4 nf_conntrack,btrfs,raid456,ip_vs
```


### 准备 CA 证书

安装 cfssl
```bash
# 从官网下载 cfssl 和 cfssljson 二进制文件，并创建软连接
root@master:~/certs# wget https://github.com/cloudflare/cfssl/releases/download/v1.6.2/cfssl_1.6.2_linux_amd64
root@master:~/certs# wget https://github.com/cloudflare/cfssl/releases/download/v1.6.2/cfssljson_1.6.2_linux_amd64

root@master:~/certs# ln -s /usr/local/bin/cfssl_1.6.2_linux_amd64 /usr/local/bin/cfssl
root@master:~/certs# ln -s /usr/local/bin/cfssljson_1.6.2_linux_amd64 /usr/local/bin/cfssljson
root@master:~/certs# ls -l /usr/local/bin/
total 20492
lrwxrwxrwx 1 root root       38 May  5 06:14 cfssl -> /usr/local/bin/cfssl_1.6.2_linux_amd64
-rwxr-xr-x 1 root root 13328560 Apr 29 09:53 cfssl_1.6.2_linux_amd64
lrwxrwxrwx 1 root root       42 May  5 06:14 cfssljson -> /usr/local/bin/cfssljson_1.6.2_linux_amd64
-rwxr-xr-x 1 root root  7646320 Apr 29 09:54 cfssljson_1.6.2_linux_amd64
```


生成默认的证书配置文件（ca-config.json），并修改。
```bash
root@master:~/certs# cfssl print-defaults config > ca-config.json

# 修改证书配置文件
root@master:~/certs# vim ca-config.json
root@master:~/certs# cat ca-config.json
{
    "signing": {
        "default": {
            "expiry": "8760h"
        },
        "profiles": {
            "kubernetes": {
                "expiry": "8760h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth",
                        "client auth"
                ]
            },
            "client": {
                "expiry": "8760h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "client auth"
                ]
            }
        }
    }
}
```
signing: 表示该证书可以用于签发证书  
expiry: 证书的有效期  
一个证书配置文件可以包含多个 profile（证书模板），在这里我们一共有两个 profile（“kubernetes” 和 “client”）  
server auth：服务器身份验证；表示 client 可以用该 CA 对 server 提供的证书进行验证；  
client auth：客户端身份验证；表示 server 可以用该 CA 对 client 提供的证书进行验证；

生成默认的 root ca 的请求文件（ca-csr.json），并修改。
```bash
root@master:~/certs# cfssl print-defaults csr > ca-csr.json
root@master:~/certs# vim ca-csr.json
root@master:~/certs# cat ca-csr.json
{
    "CN": "ca",
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
    ],
    "ca": {
        "expiry": "8760h"
    }
}
```
key：表示证书使用的算法  
因为这个证书是作为颁发证书用的，所以没有 hosts 字段


生成 root CA
```bash
root@master:~/certs# cfssl gencert -initca ca-csr.json | cfssljson -bare ca
2023/05/05 06:45:43 [INFO] generating a new CA key and certificate from CSR
2023/05/05 06:45:43 [INFO] generate received request
2023/05/05 06:45:43 [INFO] received CSR
2023/05/05 06:45:43 [INFO] generating key: rsa-2048
2023/05/05 06:45:43 [INFO] encoded CSR
2023/05/05 06:45:43 [INFO] signed certificate with serial number 238083896319705756780671176508622548310650823848
root@master:~/certs# ls -l ca*
-rw-r--r-- 1 root root  596 May  5 06:40 ca-config.json
-rw-r--r-- 1 root root 1021 May  5 06:45 ca.csr
-rw-r--r-- 1 root root  285 May  5 06:43 ca-csr.json
-rw------- 1 root root 1675 May  5 06:45 ca-key.pem
-rw-r--r-- 1 root root 1261 May  5 06:45 ca.pem
```
