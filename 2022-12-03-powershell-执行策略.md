---
title: PowerShell 执行策略
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

Powershell 执行策略是一项安全功能，能够防止恶意脚本在本地计算机上运行，但它不会影响到任何交互式命令。

Powershell 支持的策略：

  * AllSigned: 所有的脚本（包含在本机编写脚本）都必须有受信任的发布者（Publisher）签名，如果脚本没有被签名，则会被阻止运行，并有错误提示
  * ByPass: 不阻止任何操作，也没有任何警告或提
  * Default：使用默认策略，Windows Client 操作系统则使用 Restricted， Windows Server 操作系统则使用 RemoteSigned
  * RemoteSigned：可以运行本地编写的脚本，对于从 Internet 下载的脚本则需要受信任的发布者签名
  * Restricted：阻止所有脚本运行
  * Undefined：没有应用任何策略，则自动使用默认策略
  * Unrestricted：所有脚本度可以运行，当运行非本地编写脚本时，会有警告提示

<!--more-->

所有的脚本可以被运用到不同级别（范围）

  * MachinePolicy: 对计算机上的所有用户生效（更改组策略）
  * UserPolicy：只对当前用户生效（更改组策略）
  * Process：只在当前会话中生效
  * CurrentUser: 应用到当前用户（存储到注册表中 HKEY\_CURRENT\_USER)
  * LocalMachine: 应用到当前主机上的所有用户（存储到注册表中 HKEY\_LOCAL\_MACHINE)

查看当前会话的执行策略
```powershell
PS C:\Windows\system32> Get-ExecutionPolicy
RemoteSigned
```

获取影响当前会话的所有执行策略，并按优先级排序
```powershell
PS C:\Windows\system32> Get-ExecutionPolicy -List

        Scope ExecutionPolicy
        ----- ---------------
MachinePolicy       Undefined
   UserPolicy       Undefined
      Process       Undefined
  CurrentUser       Undefined
 LocalMachine    RemoteSigned
```

或者指定范围的执行策略
```powershell
PS C:\Windows\system32> Get-ExecutionPolicy -Scope Process
Undefined
```

更改执行策略(语法)
```powershell
Set-ExecutionPolicy [-ExecutionPolicy] {AllSigned | Bypass | Default | RemoteSigned | Restricted | Undefined |
    Unrestricted} [[-Scope] {CurrentUser | LocalMachine | MachinePolicy | Process | UserPolicy}]
```

更改执行策略
```powershell
PS C:\Windows\system32> Set-ExecutionPolicy -ExecutionPolicy RemoteSigned
PS C:\&gt; Get-ExecutionPolicy
RemoteSigned
```

更改指定范围的执行策略
```powershell
PS C:\Windows\system32> Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope Process

Execution Policy Change
The execution policy helps protect you from scripts that you do not trust. Changing the execution policy might expose
you to the security risks described in the about_Execution_Policies help topic at
https:/go.microsoft.com/fwlink/?LinkID=135170. Do you want to change the execution policy?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "N"): A
PS C:\Windows\system32> Get-ExecutionPolicy -List

        Scope ExecutionPolicy
        ----- ---------------
MachinePolicy       Undefined
   UserPolicy       Undefined
      Process          Bypass
  CurrentUser       Undefined
 LocalMachine    RemoteSigned

PS C:\Windows\system32> Get-ExecutionPolicy
Bypass
```