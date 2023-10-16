---
title: "Jenkinsfile 语法"
thumbnailImagePosition: left
thumbnailImage: https://img.freepik.com/free-photo/software-programming-web-development-concept_53876-176752.jpg
date: 2023-09-01
categories:
- Jenkins
- CI/CD
tags:
- Jenkins
- CI/CD
- Jenkinsfile
---

这是我们昨天的 Pipeline 项目的示例，今天我们就通过它来讲解一下 Jenkinsfile 语法。

<!--more-->

```shell
pipeline {
    agent any 
    stages {
        stage('Stage 1') {
            steps {
                sh 'echo Hello world!' 
            }
        }
        stage('Stage 2') {
            steps {
                sh 'echo I am Aaron.' 
            }
        }
    }
}
```

Jenkinsfile 通过 “{}” 来实现分层，将整个文件拆分成多个语句块。

pipeline：包含了 pipeline 需要执行的所有指令（块）。
- agent：用来表示该 pipeline 在哪个 Jenkins 节点或 workspace 上执行。其值可以为
  - any：任何可用节点
  - none：在语句块的顶层不指定任何节点，将在 stage 语句块中指定。
  - label：在拥有指定标签的节点上执行，其语法格式：agent {label “LABEL_NAME"}
  - node：在指定的 node 上运行，同样也可以使用 label 来指定 node。还可以通过 node 来实现在指定的 workspace 上执行
  - docker：在指定的容器上执行 pipeline。
- stages：处理 pipeline 的具体步骤（流程）
  - stage：具体的流程步骤。通常情况下，一个 stages 会有多个 stage。在 stage 后面有一个 ()，我们可以在里面添加 stage 的说明。
    - steps：将每个步骤，可以拆分多个具体的操作细节(step)

更多语法细节：
https://www.jenkins.io/doc/book/pipeline/syntax/