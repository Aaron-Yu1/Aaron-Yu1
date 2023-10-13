---
title: 在 Ubuntu 22.04 上安装 GitLab
thumbnailImagePosition: left
thumbnailImage: https://img.freepik.com/free-photo/coding-man_1098-18084.jpg
date: 2023-04-13
categories:
- GitLab
- CI/CD
tags:
- GitLab
- CI/CD
---

今天我们来看一看，如何在 Ubuntu 上安装 GitLab。

<!--more-->

```bash
root@jenkins:~# wget https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/ubuntu/pool/jammy/main/g/gitlab-ce/gitlab-ce_15.9.8-ce.0_amd64.deb
```

```bash
root@jenkins:~# dpkg -i gitlab-ce_15.9.8-ce.0_amd64.deb 
Selecting previously unselected package gitlab-ce.
(Reading database ... 73320 files and directories currently installed.)
Preparing to unpack gitlab-ce_15.9.8-ce.0_amd64.deb ...
Unpacking gitlab-ce (15.9.8-ce.0) ...
Setting up gitlab-ce (15.9.8-ce.0) ...
It looks like GitLab has not been configured yet; skipping the upgrade script.

       *.                  *.
      ***                 ***
     *****               *****
    .******             *******
    ********            ********
   ,,,,,,,,,***********,,,,,,,,,
  ,,,,,,,,,,,*********,,,,,,,,,,,
  .,,,,,,,,,,,*******,,,,,,,,,,,,
      ,,,,,,,,,*****,,,,,,,,,.
         ,,,,,,,****,,,,,,
            .,,,***,,,,
                ,*,.
  


     _______ __  __          __
    / ____(_) /_/ /   ____ _/ /_
   / / __/ / __/ /   / __ `/ __ \
  / /_/ / / /_/ /___/ /_/ / /_/ /
  \____/_/\__/_____/\__,_/_.___/
  

Thank you for installing GitLab!
GitLab was unable to detect a valid hostname for your instance.
Please configure a URL for your GitLab instance by setting `external_url`
configuration in /etc/gitlab/gitlab.rb file.
Then, you can start your GitLab instance by running the following command:
  sudo gitlab-ctl reconfigure

For a comprehensive list of configuration options please see the Omnibus GitLab readme
https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/README.md

Help us improve the installation experience, let us know how we did with a 1 minute survey:
https://gitlab.fra1.qualtrics.com/jfe/form/SV_6kVqZANThUQ1bZb?installation=omnibus&release=15-9

```


```bash
root@jenkins:~# vim /etc/gitlab/gitlab.rb
root@jenkins:~# cat /etc/gitlab/gitlab.rb | grep -Ev '^$|#' | grep -e ^external_url -e initial_root_password
external_url 'http://gitlba.test.local'
gitlab_rails['initial_root_password'] = "P@ssw0rd"
```


```bash
root@jenkins:~# gitlab-ctl reconfigure
```

打开浏览器输入前面的 external_url 地址，并输入用户名密码，进入 gitlab。用户名为 root，密码为前面的 initial_root_password 中配置的


安装脚本
```bash
#/bin/bash

# 定义脚本需要使用的变量
# protocol 为 external_url 配置中，外部访问使用的协议，一般为 http 或 https
# 目前脚本只支持 http，https 还没有测试
# fqdn 为 external_url 使用的主机名
# password 为 root 用户的初始密码，用于第一次登录 gitlab，建议登录后更改
protocol=http
fqdn='gitlab.test.local'
password="P@ssw0rd"

# 下载安装包
# 根据需要选择合适的版本，以及安装包的下载地址
wget https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/ubuntu/pool/jammy/main/g/gitlab-ce/gitlab-ce_15.9.8-ce.0_amd64.deb

# 安装软件包
dpkg -i gitlab-ce_15.9.8-ce.0_amd64.deb

# 更改配置
# external_url 为外部访问 URL 地址
sed -i "s#external_url.*#external_url \'$protocol:\/\/$fqdn\'#" /etc/gitlab/gitlab.rb
sed -i "s/.*gitlab_rails.*'initial_root_password.*/gitlab_rails[\'initial_root_password\'] = \"$password\"/" /etc/gitlab/gitlab.rb

# 初始化
gitlab-ctl reconfigure
```