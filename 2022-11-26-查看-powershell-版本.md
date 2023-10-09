thumbnailImagePosition: left
thumbnailImage: https://img.freepik.com/premium-photo/digital-marketing-media-social-network-mobile-app-virtual-icon-diagram-business-using-smartphone-tablet-as-concept_533878-307.jpg
coverImage: https://img.freepik.com/premium-photo/digital-marketing-technology-concept-online-search-engine-social-network-financial-banking_34200-848.jpg
metaAlignment: center
coverMeta: out
date: 2022-12-03
categories:
- PowerShell
tags:
- PowerShell

---

我们可以通过 $PSVersionTable 变量或者 Get-Host 命令来查看 Powershell 当前的版本。

<!--more-->

```powershell
PS C:\Users\Admin> $PSVersionTable

Name                           Value
----                           -----
PSVersion                      5.1.19041.1682
PSEdition                      Desktop
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0...}
BuildVersion                   10.0.19041.1682
CLRVersion                     4.0.30319.42000
WSManStackVersion              3.0
PSRemotingProtocolVersion      2.3
SerializationVersion           1.1.0.1

PS C:\Users\Admin> Get-Host

Name             : ConsoleHost
Version          : 5.1.19041.1682
InstanceId       : 6db9d4be-3432-4b9a-80e8-9f99b4f0111d
UI               : System.Management.Automation.Internal.Host.InternalHostUserInterface
CurrentCulture   : en-US
CurrentUICulture : en-US
PrivateData      : Microsoft.PowerShell.ConsoleHost+ConsoleColorProxy
DebuggerEnabled  : True
IsRunspacePushed : False
Runspace         : System.Management.Automation.Runspaces.LocalRunspace
```