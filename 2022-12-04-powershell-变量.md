---
title: PowerShell 变量
thumbnailImagePosition: left
thumbnailImage: https://img.freepik.com/premium-photo/business-screen-virtual-technology-digital-futuristic-display-innovation-concept-interface-tech-hand_163305-212005.jpg
coverImage: https://img.freepik.com/free-vector/vector-infographic-background-with-cartoon-interior-future-data-center-room-with-server-hardware-hologram-processor-concept-bigdata-technology-cloud-information-base_107791-3567.jpg
metaAlignment: center
coverMeta: out
date: 2022-12-04
categories:
- PowerShell
tags:
- PowerShell
---
在 PowerShell 中，你可以使用变量来存储所有类型的值，如，数字，字符串，命令的结果，等等。

在 PowerShell 中，变量以美元符号开头，如，$a, $process，并且变量名称不区分大小写，即 $profile 与 $PROFILE 是同一个变量。

Powershell 变量名可以是任何字符，并且不区分大小写。但是有些命名方式虽然支持，但不推荐使用,如，以特殊字符开头，包含空格等等

<!--more-->

```powershell
PS C:\Windows\system32> ${@1}=4
PS C:\Windows\system32> ${@1}
4
```

我们可以通过 “=” 符号来给变量赋值，没有被赋值的变量的值为 $null。
```powershell
PS C:\Windows\system32> $logPath = "C:\Logs\"
PS C:\Windows\system32> echo $logPath
C:\Logs\
```

删除变量的值

```powershell
PS C:\Windows\system32> Get-Item -Path Variable:\logPath

Name                           Value
----                           -----
logPath                        C:\Logs\

PS C:\Windows\system32> Clear-Variable -Name logPath
PS C:\Windows\system32> $logPath
PS C:\Windows\system32> Get-Item -Path Variable:\logPath

Name                           Value
----                           -----
logPath
```

删除变量
```powershell
PS C:\Windows\system32> Remove-Variable -Name logPath
PS C:\Windows\system32> $logPath
PS C:\Windows\system32> Get-Item -Path Variable:\logPath
Get-Item : Cannot find path 'Variable:\logPath' because it does not exist.
At line:1 char:1
+ Get-Item -Path Variable:\logPath
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (Variable:\logPath:String) [Get-Item], ItemNotFoundException
    + FullyQualifiedErrorId : PathNotFound,Microsoft.PowerShell.Commands.GetItemCommand
```

注释：既然我们可以通过 Get-Item 来查看变量，那么我们也可以通过 Remove-Item 来删除变量。

变量的引用（'' 和 “ ”）：
```powershell
PS C:\Windows\system32> $value = "string"
PS C:\Windows\system32> "This is $value"
This is string
PS C:\Users\Administrator> 'This is $value'
This is $value
```

转义字符 (\`)

```powershell
PS C:\Windows\system32> "The value of $value is $value"
The value of string is string
PS C:\Windows\system32> "The value of `$value is $value"
The value of $value is string
```