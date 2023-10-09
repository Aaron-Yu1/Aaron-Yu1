---
title: Powershell Profile
thumbnailImagePosition: left
thumbnailImage: https://img.freepik.com/free-photo/html-css-collage-concept-with-person_23-2150062010.jpg
coverImage: https://img.freepik.com/free-photo/freelance-young-asian-businesswoman-casual-wear-using-laptop-working-living-room-home_7861-3022.jpg
metaAlignment: center
coverMeta: out
date: 2022-12-03
categories:
- PowerShell
tags:
- PowerShell
---
我们可以通过 Profile 来自定义我们的 PowerShell 环境，如，自定义命令的别名，默认的工作路径，等等

PowerShell 支持多个 Profile（优先级从高到底）：

  * 所有用户，所有主机
  * 所有用户，当前主机
  * 当前用户，所有主机
  * 当前用户，当前主机

<!--more-->

```powershell
PS C:\Windows\system32> $PROFILE
C:\Users\Administrator\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1   # 这个文件默认不存

PS C:\Windows\system32> $PROFILE | fl -Force

AllUsersAllHosts       : C:\Windows\System32\WindowsPowerShell\v1.0\profile.ps1
AllUsersCurrentHost    : C:\Windows\System32\WindowsPowerShell\v1.0\Microsoft.PowerShell_profile.ps1
CurrentUserAllHosts    : C:\Users\Administrator\Documents\WindowsPowerShell\profile.ps1
CurrentUserCurrentHost : C:\Users\Administrator\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1
Length                 : 83</code></pre>
```

新建 Profile 文件

```powershell
PS C:\Windows\system32> Test-Path -Path $PROFILE.CurrentUserAllHosts
False
PS C:\Windows\system32> New-Item -ItemType File $PROFILE.CurrentUserAllHosts

    Directory: C:\Users\Administrator\Documents\WindowsPowerShell

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----       11/17/2022   4:17 PM              0 profile.ps1
```

编辑新文件内容（这些内容都会在启动 PowerShell 的时候运行，并应用）：
```powershell
New-Alias -Name Connect -Value Enter-PSSession

$ComputerName = "192.168.1.100","192.168.1.101"

cd C:\

$session = New-PSSession -ComputerName "192.168.1.100"
```

重新打开 PowerShell
```powershel
PS C:\> $ComputerName
192.168.1.100
192.168.1.101
PS C:\> Connect -ComputerName "192.168.1.100"
[192.168.1.100]: PS C:\Users\Administrator\Documents&gt; exit
PS C:\> $session

 Id Name            ComputerName    ComputerType    State         ConfigurationName     Availability
 -- ----            ------------    ------------    -----         -----------------     ------------
  1 WinRM1          192.168.1.100   RemoteMachine   Opened        Microsoft.PowerShell     Available
```
