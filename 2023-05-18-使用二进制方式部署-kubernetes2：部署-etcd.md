---
title: 使用二进制方式部署 kubernetes(2)：部署 etcd
thumbnailImagePosition: left
thumbnailImage: https://img.freepik.com/free-vector/api-concept-illustration_114360-9822.jpg
date: 2023-05-18
categories:
- Kubernetes
- etc
tags:
- Kubernetes
- etcd
---

前面，我们已经完成的每个节点的初始化配置，以及生成了 root CA，今天我们来看看，通过 root CA 为 etcd 准备证书，以及通过二进制的方式部署 etcd。

<!--more-->

首先，我们需要为 etcd 创建证书请求，并将 etcd 服务器地址加入到请求的 hosts 字段中。
```bash
root@master:~/certs# vim etcd-csr.json
root@master:~/certs# cat etcd-csr.json
{
    "CN": "etcd",
    "hosts": [
        "10.0.0.99",
        "10.0.0.176",
        "10.0.2.169",
        "127.0.0.1"
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

生成 ectd 证书
```bash
root@master:~/certs# cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes etcd-csr.json | cfssljson -bare etcd
2023/05/05 06:59:38 [INFO] generate received request
2023/05/05 06:59:38 [INFO] received CSR
2023/05/05 06:59:38 [INFO] generating key: rsa-2048
2023/05/05 06:59:39 [INFO] encoded CSR
2023/05/05 06:59:39 [INFO] signed certificate with serial number 224162062411301274304963811044615691702573023528
root@master:~/certs# ls -l etcd*
-rw-r--r-- 1 root   root      1050 May  5 06:59 etcd.csr
-rw-r--r-- 1 root   root       355 May  5 06:58 etcd-csr.json
-rw------- 1 root   root      1675 May  5 06:59 etcd-key.pem
-rw-r--r-- 1 root   root      1395 May  5 06:59 etcd.pem
```

我们需要为每个节点准备一个目录，用于存储证书文件，然后将生成好的 etcd 证书复制到该目录。
```bash

root@master:~/certs# mkdir -p /etc/kubernetes/{etcd,ssl}
root@master:~/certs# cp {etcd.pem,etcd-key.pem,ca.pem} /etc/kubernetes/ssl/
root@master:~/certs# ls /etc/kubernetes/ssl/
ca.pem  etcd-key.pem  etcd.pem
```

安装 etcd
```bash
# 下载 etcd 的二进制源码包，并解压
root@master:~/pkg# wget https://github.com/etcd-io/etcd/releases/download/v3.5.6/etcd-v3.5.6-linux-amd64.tar.gz
root@master:~/pkg# tar -zxf etcd-v3.5.6-linux-amd64.tar.gz

# 复制 etcd 和 etcdctl 到 /usr/local/bin/ 目录
# etcdctl 是 etcd 的命令行工具，复制到 /usr/local/bin/ 目录是为了不需要修改 $PATH 环境变量
root@master:~/pkg# cp etcd-v3.5.6-linux-amd64/{etcd,etcdctl} /usr/local/bin/

# 创建 etcd 服务器文件
root@master:~/certs# vim /usr/lib/systemd/system/etcd.service
root@master:~/certs# cat /usr/lib/systemd/system/etcd.service 
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target
Documentation=https://github.com/coreos

[Service]
Type=notify
WorkingDirectory=/etc/kubernetes/etcd
ExecStart=/usr/local/bin/etcd \
  --advertise-client-urls=https://10.0.0.99:2379 \
  --cert-file=/etc/kubernetes/ssl/etcd.pem \
  --client-cert-auth=true \
  --data-dir=/etc/kubernetes/etcd/data \
  --experimental-initial-corrupt-check=true \
  --experimental-watch-progress-notify-interval=5s \
  --initial-advertise-peer-urls=https://10.0.0.99:2380 \
  --initial-cluster="etcd-master=https://10.0.0.99:2380,etcd-work1=https://10.0.0.176:2380,etcd-work2=https://10.0.2.169:2380" \
  --initial-cluster-state=new \
  --initial-cluster-token=etcd-cluster \
  --key-file=/etc/kubernetes/ssl/etcd-key.pem \
  --listen-client-urls=https://10.0.0.99:2379,https://127.0.0.1:2379 \
  --listen-peer-urls=https://10.0.0.99:2380 \
  --name=etcd-master \
  --peer-cert-file=/etc/kubernetes/ssl/etcd.pem \
  --peer-client-cert-auth=true \
  --peer-key-file=/etc/kubernetes/ssl/etcd-key.pem \
  --peer-trusted-ca-file=/etc/kubernetes/ssl/ca.pem \
  --snapshot-count=50000 \
  --trusted-ca-file=/etc/kubernetes/ssl/ca.pem

[Install]
WantedBy=multi-user.target
```
--data-dir：数据库的存储目录，建议与 --wal-dir 分开，我这里是实验环境，所以没有单独指定 --wal-dir；  
这是一个新的群集，所以 --initial-cluster-state 的值为 new  
--name：该节点的名称，并且该值将会被 --initial-cluster 引用
完整参数列表可以通过 etcd --help 查询  

复制所需的文件到其它两个节点：
```bash
root@master:~/certs# scp /usr/local/bin/{etcd,etcdctl} 10.0.0.176:/usr/local/bin
root@master:~/certs# scp /usr/local/bin/{etcd,etcdctl} 10.0.2.169:/usr/local/bin
root@master:~/certs# scp /usr/lib/systemd/system/etcd.service 10.0.0.176:/usr/lib/systemd/system/
root@master:~/certs# scp /usr/lib/systemd/system/etcd.service 10.0.2.169:/usr/lib/systemd/system/
root@master:~/certs# scp /etc/kubernetes/ssl/{ca.pem,etcd.pem,etcd-key.pem} 10.0.0.176:/etc/kubernetes/ssl/   
root@master:~/certs# scp /etc/kubernetes/ssl/{ca.pem,etcd.pem,etcd-key.pem} 10.0.2.169:/etc/kubernetes/ssl/
```

在其他节点中更改 service 文件的 IP 信息，以及节点的名称
```bash
root@work1:~# vim /usr/lib/systemd/system/etcd.service
root@work2:~# vim /usr/lib/systemd/system/etcd.service
```

在所有节点启动服务（在启动第一台 etcd 主机后，就需要尽快启动其它两台主机上的 etcd service）
```bash
root@master:~/certs# systemctl daemon-reload 
root@master:~/certs# systemctl enable --now etcd.service
root@master:~/certs# systemctl status etcd.service 
● etcd.service - Etcd Server
     Loaded: loaded (/lib/systemd/system/etcd.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2023-05-05 07:21:08 CST; 1min 55s ago
       Docs: https://github.com/coreos
   Main PID: 191952 (etcd)
      Tasks: 8 (limit: 4619)
     Memory: 26.1M
     CGroup: /system.slice/etcd.service
             └─191952 /usr/local/bin/etcd --advertise-client-urls=https://10.0.0.99:2379 --cert-file=/etc/kubernet>

May 05 07:21:16 master etcd[191952]: {"level":"info","ts":"2023-05-05T07:21:16.678+0800","caller":"rafthttp/peer_s>
May 05 07:21:16 master etcd[191952]: {"level":"info","ts":"2023-05-05T07:21:16.684+0800","caller":"rafthttp/stream>
May 05 07:21:16 master etcd[191952]: {"level":"info","ts":"2023-05-05T07:21:16.684+0800","caller":"rafthttp/stream>
May 05 07:21:16 master etcd[191952]: {"level":"info","ts":"2023-05-05T07:21:16.684+0800","caller":"rafthttp/stream>
May 05 07:21:16 master etcd[191952]: {"level":"info","ts":"2023-05-05T07:21:16.684+0800","caller":"rafthttp/stream>
May 05 07:21:16 master etcd[191952]: {"level":"info","ts":"2023-05-05T07:21:16.696+0800","caller":"rafthttp/stream>
May 05 07:21:16 master etcd[191952]: {"level":"info","ts":"2023-05-05T07:21:16.697+0800","caller":"rafthttp/stream>
May 05 07:21:20 master etcd[191952]: {"level":"info","ts":"2023-05-05T07:21:20.502+0800","caller":"etcdserver/serv>
May 05 07:21:20 master etcd[191952]: {"level":"info","ts":"2023-05-05T07:21:20.504+0800","caller":"membership/clus>
May 05 07:21:20 master etcd[191952]: {"level":"info","ts":"2023-05-05T07:21:20.504+0800","caller":"etcdserver/serv>
```

验证
```bash
root@master:~/certs# etcdctl --cacert=/etc/kubernetes/ssl/ca.pem --cert=/etc/kubernetes/ssl/etcd.pem --key=/etc/kubernetes/ssl/etcd-key.pem --endpoints=https://10.0.0.99:2379,https://10.0.0.176:2379,https://10.0.2.169:2379 endpoint health
https://10.0.0.99:2379 is healthy: successfully committed proposal: took = 10.966898ms
https://10.0.2.169:2379 is healthy: successfully committed proposal: took = 13.822152ms
https://10.0.0.176:2379 is healthy: successfully committed proposal: took = 14.347266ms
```