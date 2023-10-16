---
title: "Jenkins 中的环境变量"
thumbnailImagePosition: left
thumbnailImage: https://s1.imagehub.cc/images/2023/10/09/left1.jpeg
date: 2023-09-01
categories:
- Jenkins
- CI/CD
tags:
- Jenkins
- CI/CD
- Environment vriables
- 全局环境变量
- 自定义环境变量
- 动态环境变量
---

前面，我们已经讲解了 Jenkinsfile 的基本语法，今天我们来看一看 Jenkinsfile 中的环境变量。

<!--more-->

在 Jenkins 中，环境变量（environment vriables）可分为：
- 全局环境变量
- 自定义环境变量
- 动态环境变量


#### 全局环境变量：
全局环境变量，就是 Jenkins 的内置公开的变量，你可以在 Pipeline 文件中，直接使用它。

你可以在这个连接里面获取到完成的全局环境变量列表以及说明：
${YOUR_JENKINS_URL}/pipeline-syntax/globals#env 

我们这里就随便选择了两个作为说明：
    BUILD_ID：就是我们前面说的 Build 名，它由 # 加 数字构成。
    $NODE_NAME：构建节点的名称，如果是直接在 Jenkins 上构建的，其值为 “built-in”，如果在代理节点上构建，其值为代理节点的名称

使用此 pipeline 文件创建一个 pipeline 项目，并进行 build。
```bash
pipeline {
    agent any
    stages {
        stage('echo') {
            steps {
                sh 'echo $BUILD_ID on $NODE_NAME'
            }
        }
    }
}
```
![1](images/1.png)

Build 完成后，你可以在 Build History 下面看到一个数字，第一次 Build，这个值应该就是 1，这个就是 Build ID。点击这个 ID，进入这个 Build。
![2](images/2.png)

在 Console Output 你可以看到 “1 on built-in” 这样的输出，其中 “1” 就是你的 BUILD_ID，“built-in” 表示你是在 Jenkins 上构建的。
![3](images/3.png)

除了这些内置你全局环境变量，我们还可以自定义全局环境变量。
点击 Manage Jenkins > System，找到 Global properties，勾选 Environment Variables。然后点击 Add。

输入 Name 和 Value。可以再次点击 Add 按钮，添加新的环境变量。
    Name: git_project
    Value: https://github.com/Aaron-Yu1/mysite_django

在 pipeline 中引用。   
```bash
pipeline {
    agent any
    stages {
        stage('echo') {
            steps {
                sh 'echo ${git_project}'
            }
        }
    }
}
```

#### 自定义变量
顾名思义，就是我们自己定义的变量。

我们可以通过 environment “关键字” 来组织一个环境变量语句块，在这个语句块中定义我们需要使用的环境变量。如下面的示例，我们定义了两个变量，FIRSTNAME 和 LASTNAME。然后在 echo 中输出这两个变量的值。

修改我们前面创建的 pipeline 项目，并重新 build。
```bash
pipeline {
    agent any
    environment {
        FIRSTNAME = 'Aaron'
        LASTNAME = 'Yu'
    }
    stages {
        stage('echo') {
            steps {
                sh 'echo $FIRSTNAME $LASTNAME'
            }
        }
    }
}
```
![4](images/4.png)

Build 完成后，我们可以看到 build ID 变成了 #2，点击该 build。我们可以看到我们自定义的环境变量在 Console Output 中输出了。
![5](images/5.png)

#### 动态环境变量
就是将脚本（命令）的输出的内容或返回的状态码返回赋值给变量。本质上和自定义变量没有什么区别。其中 returnStatus 表示返回的是状态码， returnStdout 表示返回的是输出的内容。

修改我们前面创建的 pipeline 项目，并重新 build。
```bash
pipeline {
    agent any 
    environment {
        CC = """${sh(
                returnStdout: true,
                script: 'echo "clang"'
            )}""" 
        EXIT_STATUS = """${sh(
                returnStatus: true,
                script: 'exit 1'
            )}"""
    }
    stages {
        stage('Example') {
            steps {
                sh 'echo $CC'
                sh 'echo $EXIT_STATUS'
            }
        }
    }
}
```
![6](images/6.png)

Build 完成后，点击对应的 Build ID。我们可以看到这些环境变量在 Console Output 中输出了。
![7](images/7.png)