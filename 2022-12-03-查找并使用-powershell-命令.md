---
title: 查找并使用 PowerShell 命令
thumbnailImagePosition: left
thumbnailImage: https://img.freepik.com/premium-photo/techsavvy-professional-immersed-virtual-realm-surrounded-by-multiple-screens-aigenerated-stock_561855-16851.jpg
coverImage: https://img.freepik.com/premium-photo/business-people-advisor-business-people-talking-planning-analyze-investment-marketing-tablet-office_44344-3887.jpg
metaAlignment: center
coverMeta: out
date: 2022-12-03
categories:
- PowerShell
tags:
- PowerShell
---

前面我们有介绍通过 Get-Help 或 Help 去获取命令的帮助，但在实际使用的时候，我们只看那些帮助是不够的，尤其是在写脚本的时候，所以这里介绍一些方法以及其他的命令，来帮助我们更好地使用 powershell 命令

Tab 键自定补全  
在很多情况我们可能会忘记命令的全称或命令太长，逐个输入可能会有输错的情况，此时我们可以通过 tab 键来自定补全命令。如果存在多个命令的可能，Powershell 会按字母的顺序，向你逐个展示它们。

<!--more-->
```Powershell
PS C:\Windows\system32> get-ser<tab>
```


使用 tab 键补全，你必须要知道命令的开头，如果你只记得命令的部分，如，我只记得命令中包含了 service，可以通过下面的命令获取帮助，它会列出所有包含 service 的命令

```Powershell
PS C:\Windows\system32> Get-Command -Name "*Service*"
PS C:\&gt; Get-Command -Name "*User*" -Module ActiveDirectory  # 从指定的模块中查找
PS C:\&gt; Get-Command -Verb Set -Noun *user*    # 查找动词部分为 Set，名词部包含 User 的命令
```
说明：Powershell 命令由 “Verb-Noun（动词-名词）构成，如，Get-Service, Get-Process, Stop-Service 等等

当我们知道了命令的使用方法后，对于命令的输出，我想知道命令支持那些属性和方法，我们可以通过 “ | Get-Member” 去查看

```Powershell
PS C:\Windows\system32> Get-Service | Get-Member

   TypeName: System.ServiceProcess.ServiceController

Name                      MemberType    Definition
----                      ----------    ----------
Name                      AliasProperty Name = ServiceName
RequiredServices          AliasProperty RequiredServices = ServicesDependedOn
Disposed                  Event         System.EventHandler Disposed(System.Object, System.EventArgs)
Close                     Method        void Close()
Continue                  Method        void Continue()
CreateObjRef              Method        System.Runtime.Remoting.ObjRef CreateObjRef(type requestedType)
Dispose                   Method        void Dispose(), void IDisposable.Dispose()
Equals                    Method        bool Equals(System.Object obj)
ExecuteCommand            Method        void ExecuteCommand(int command)
GetHashCode               Method        int GetHashCode()
GetLifetimeService        Method        System.Object GetLifetimeService()
... ...
... ...
```