---
title: "通过 Webhook 触发 Jenkins Build"
thumbnailImagePosition: left
thumbnailImage: https://s1.imagehub.cc/images/2023/10/09/left1.jpeg
date: 2023-09-05
categories:
- Jenkins
- CI/CD
tags:
- Jenkins
- CI/CD
---




登录到 GitHub 创建一个 token。勾选 repo 和 admin:repo_hook. Token 创建好了之后最好复制一份放到一个文本文件中，等配置好了再删除，因为 Token 只出现一次，当你关掉之后，再去查看，是没法看到 Token 的具体内容的。

然后导航到你需要配置的 GitHub 项目中，点击当前项目的 Settings，然后找到 Webhooks，点击 Add Webhooks 按钮，添加一个新的 webhooks。输入以下信息：
- Payload URL: <Jenkins URL>/github-webhook/
- Content type: Application/json
- Secret: Jenkins admin 的 token，上一章有介绍过。
- 勾选：
    - Just the push event
    - Active
然后点击 Add Webhooks。
A1

在添加 Payload URL 时，最后的 / 不要忘了。


在 Jenkins 中，使用前面在 GitHub 上生成的 token 为 GitHub 创建一个凭据。
71

导航到 Manage Jenkins > System > GitHub > GitHub Servers。点击 Add GitHub Server，添加 GitHub 服务。
72

在 Name 文本窗口，输入 GitHub Server 的名称（自定义），在 API URL 输入 GitHub 的 API URL，一般情况默认即可，Credentials 选择我们前面使用 GitHub token 创建的凭据即可。然后右下角的 test connection，测试连接。
73

当看到下面信息，表示测试连接成功。然后点击下面的 Save 按钮，保存配置。
74

使用下面 pipeline 脚本，创建一个 project
```bash
pipeline {
    agent any
    stages {
        stage('Pull') {
            steps {
                retry(3) {
                    git branch: 'master', url: 'https://github.com/Aaron-Yu1/jenkins-test.git'
                }
            }
        }
        stage('Test') {
            steps {
                sh 'echo pylint --rcfile=pylint.conf base'
                sh 'echo Unit Testing...'
            }
        }
        stage('Build') {
            steps {
                sh ''' 
                    echo building...
                '''
            }
        }
    }
    post { 
        cleanup {
            sh '''
                rm -rf ./*
                echo docker builder prune -f
            '''
        }
    }
}
```
同时勾选：
- GitHub project
- GitHub hook trigger for GITScm polling
75

然后尝试手动进行 Build，Build 成功后，修改 GitHub 项目，并 push。

push 完成后，回到 GitHub 的 webhooks 页面，并点击到你所添加的 webhooks，在 Recent Deliveries 页面，你会看到 GitHub 向 Jenkins 服务器推送的信息。
A2

回到 Jenkins 上面的项目，你会看到一个新的 Build 完成（具体情况取决于内 Pipeline 脚本的内容）。点击该 Build ID，在 Status 页面，你会看到 “Started by GitHub push by ...".
76