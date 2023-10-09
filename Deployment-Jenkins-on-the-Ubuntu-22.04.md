---
title: "在 Ubuntu 22.04 上部署 Jenkins"
thumbnailImagePosition: left
thumbnailImage: https://s1.imagehub.cc/images/2023/10/09/left1.jpeg
coverImage: //d1u9biwaxjngwg.cloudfront.net/cover-image-showcase/city.jpg
metaAlignment: center
coverMeta: out
date: 2023-10-01
categories:
- Jenkins
- CI/CD
tags:
- Jenkins
- CI/CD
---

安装 OpenJDK。 安装 jdk 需要注意当前安装的 Jenkins 是否支持所安装的 JDK 版本。

<!--more-->

```bash
root@jenkins:~# apt update
root@jenkins:~# apt install -y openjdk-17-jdk
```

```bash
root@jenkins:~# curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
root@jenkins:~# echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | tee  /etc/apt/sources.list.d/jenkins.list > /dev/null
root@jenkins:~# apt update
root@jenkins:~# apt-get install -y jenkins
```

验证服务
```bash
root@jenkins:~# systemctl status jenkins.service 
● jenkins.service - Jenkins Continuous Integration Server
     Loaded: loaded (/lib/systemd/system/jenkins.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2023-09-19 02:29:06 UTC; 3min 5s ago
   Main PID: 99091 (java)
      Tasks: 48 (limit: 2122)
     Memory: 704.9M
        CPU: 1min 40.088s
     CGroup: /system.slice/jenkins.service
             └─99091 /usr/bin/java -Djava.awt.headless=true -jar /usr/share/java/jenkins.war --webroot=/var/cache/>

Sep 19 02:28:26 jenkins jenkins[99091]: 2303c301892c41cabd37eb3d19a41c1f
Sep 19 02:28:26 jenkins jenkins[99091]: This may also be found at: /var/lib/jenkins/secrets/initialAdminPassword
Sep 19 02:28:26 jenkins jenkins[99091]: *************************************************************
Sep 19 02:28:26 jenkins jenkins[99091]: *************************************************************
Sep 19 02:28:26 jenkins jenkins[99091]: *************************************************************
Sep 19 02:29:06 jenkins jenkins[99091]: 2023-09-19 02:29:06.556+0000 [id=31]        INFO        jenkins.InitReacto>
Sep 19 02:29:06 jenkins jenkins[99091]: 2023-09-19 02:29:06.584+0000 [id=24]        INFO        hudson.lifecycle.L>
Sep 19 02:29:06 jenkins systemd[1]: Started Jenkins Continuous Integration Server.
Sep 19 02:29:07 jenkins jenkins[99091]: 2023-09-19 02:29:07.760+0000 [id=48]        INFO        h.m.DownloadServic>
Sep 19 02:29:07 jenkins jenkins[99091]: 2023-09-19 02:29:07.761+0000 [id=48]        INFO        hudson.util.Retrie>
```


获取 Jenkins 初始密码，进行初始化。
```bash
root@jenkins:~# cat /var/lib/jenkins/secrets/initialAdminPassword
2303c301892c41cabd37eb3d19a41c1f
```

在浏览器地址栏中输入 htt://<Jenkins IP\>:8080, 打开 Jenkins 初始化页面。输入初始化密码，初始化 Jenkins。
![A](https://s1.imagehub.cc/images/2023/10/09/A.png)

安装推荐的插件（Suggested Plugins）。你也可以通过 Select plugins to install 选项来自定义需要安装的插件。
![B](https://s1.imagehub.cc/images/2023/10/09/B.png)

等待插件安装完成。
![C](https://s1.imagehub.cc/images/2023/10/09/C.png)

创建第一个 admin 用户，需要提供以下信息：
- Username：用户名
- Password：用户密码
- Full Name：完整的名称
- E-mail address：邮件地址
![D](https://s1.imagehub.cc/images/2023/10/09/D.png)

实例（Instance）配置
![E](https://s1.imagehub.cc/images/2023/10/09/E.png)

配置完成，可以开始使用 Jenkins 了。
![F](https://s1.imagehub.cc/images/2023/10/09/F.png)