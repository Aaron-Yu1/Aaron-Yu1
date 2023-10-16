---
title: "Jenkins 中的 Post"
thumbnailImagePosition: left
thumbnailImage: https://img.freepik.com/premium-photo/programming-power-laptop-code-generative-ai_893610-4587.jpg
date: 2023-09-01
categories:
- Jenkins
- CI/CD
tags:
- Jenkins
- CI/CD
- Post
---

通过前面的示例，我们可以看到每个 pipeline 或 stage 运行结束后，都会返回一个结果（Finished 的状态）。post 的作用就是让我们可以将这个返回状态的值作为条件，决定运行一些额外的指令。如，一些可能预知的错误，在这里添加某些指令来修复。

<!--more-->

post 可以支持以下判断条件：
- always：不管结果如何，总是运行该部分
- changed：只有当当前返回状态与之前不同时，才会运行该部分
- fixed：前一次运行失败，这次运行成功了，才会运行该部分
- regression：当前状态为失败，但前一次运行成功时，才会运行该部分
- aborted：当前状态为 aborted 时（用户手动终止），才会运行该部分
- failure：当前状态为 failure 时，才会运行该部分
- unstable：当前状态为 unstable 时，才会运行该部分
- success：当前状态为 success 时，才会运行该部分
- unsuccessful：当前状态为 unsuccessful 时，才会运行该部分
- cleanup：它和 always 类似，不关心返回状态，总是运行，但是，区别在于它是在评估完当前 post 中的其他条件之后运行（总是最后运行）。


由于篇幅问题，我就不截图了，直接写每次的结果，并符上 pipeline 脚本。

Always: Build 后，我们发现结果是 “SUCCESS”，并且两条指令都执行了。我们可以将第一条指令中的 “echo” 改成 “cho”，并再次 Build，我们会发现 Build 失败了，其返回结果是 “FAILURE”，并且第一条指令没有被执行，但是第二条指令依然被执行了。
```bash
pipeline {
    agent any
    stages {
        stage('test') {
            steps {
                sh 'echo Hello World'
            }
        }
    }
    post { 
        always { 
            sh 'echo I am always run!'
        }
    }
}
```

Changed：继续前面的 job，改回 echo，并将 always 改成 changed，再次 Build。Build 成功，并返回 “SUCCESS”（状态较前一次发生改变），所以第二条指令执行了。在没有任何改动的情况下，我们再次 Build，返回的结果依然是 “SUCCESS”（状态较前一次没有发生改变），所以第二条指令没有执行。我们再次将第一条指令的 “echo” 改成 “cho”，并 Build，返回结果是 “FAILURE”（状态较前一次发生改变），第一条指令没有执行，但是第二条指令执行了。
```bash
pipeline {
    agent any
    stages {
        stage('test') {
            steps {
                sh 'echo Hello World'
            }
        }
    }
    post { 
        changed { 
            sh 'echo I will run if have changed!'
        }
    }
}
```

Fixed：修正第一条指令的 echo，并将 post 指令改成 fixed，然后进行 Build（最后一次 Build 的结果是 “FAILURE”），Build 的结果是 “SUCCESS”，并且两条指令都执行。再次将第一条指令的 “echo” 改成 “cho”，并 Build，返回结果是 “FAILURE”，并且两条指令都没有执行。
```bash
pipeline {
    agent any
    stages {
        stage('test') {
            steps {
                sh 'echo Hello World'
            }
        }
    }
    post { 
        fixed { 
            sh 'echo I will run if it is fixed!'
        }
    }
}
```

Regression：修正 echo 指令，然后进行 Build，Build 成功，返回 “SUCCESS”，但只有第一条指令执行了，第二条没有执行。将第一条指令的 “echo” 改成 “cho”，并 Build，返回结果是 “FAILURE”， 但是第二条指令执行了。不做任何更改，再次 Build，结果依然是 “FAILURE”，但是这次两条指令都没有执行。
```bash
pipeline {
    agent any
    stages {
        stage('test') {
            steps {
                sh 'echo Hello World'
            }
        }
    }
    post { 
        regression { 
            sh 'echo I will regression!'
        }
    }
}
```
regression 当我们 Build 错误时，执行代码回滚时，非常有用。


关于 aborted，failure，unstable，success 和 unsuccessful 我这里就不演示了，这些都是正常的结果返回状态，如果结果是对应的状态，则运行对应的代码块。