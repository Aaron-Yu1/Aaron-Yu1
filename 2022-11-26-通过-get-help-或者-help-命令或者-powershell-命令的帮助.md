---
title: 通过 Get-Help 或者 Help 命令或者 Powershell 命令的帮助
thumbnailImagePosition: left
thumbnailImage: https://www.freepik.com/premium-ai-image/online-cloud-storage-contact-computing-tablet-phone-home-devices-with-online-linking-computer-cloud-with-server-connection-devices-connected-storage-data-center_57711599.htm
coverImage: https://img.freepik.com/free-photo/transportation-technology-concept-intelligent-transport-systems_587448-4777.jpg
metaAlignment: center
coverMeta: out
date: 2022-12-03
categories:
- PowerShell
tags:
- PowerShell
---
Powershell 命令的信息包含：

  * Name：Powershell 命令的名称
  * Synopsis：摘要信息
  * Syntax：命令的语法
  * Description：命令的描述信息
  * Related Link：命令相关的链接（在线帮助）
  * Remarks：备注信息

<!--more-->

```powershell
PS C:\Windows\system32> Get-Help Write-Host

NAME
    Write-Host

SYNOPSIS
    Writes customized output to a host.

SYNTAX
    Write-Host [[-Object] &lt;System.Object&gt;] [-BackgroundColor {Black | DarkBlue | DarkGreen | DarkCyan | DarkRed | DarkMagenta | DarkYellow | Gray | DarkGray | Blue | Green
    | Cyan | Red | Magenta | Yellow | White}] [-ForegroundColor {Black | DarkBlue | DarkGreen | DarkCyan | DarkRed | DarkMagenta | DarkYellow | Gray | DarkGray | Blue |
    Green | Cyan | Red | Magenta | Yellow | White}] [-NoNewline] [-Separator &lt;System.Object&gt;] [&lt;CommonParameters&gt;]

DESCRIPTION
    The `Write-Host` cmdlet&#039;s primary purpose is to produce for-(host)-display-only output, such as printing colored text like when prompting the user for input in
    conjunction with Read-Host (Read-Host.md). `Write-Host` uses the ToString() (/dotnet/api/system.object.tostring)method to write the output. By contrast, to output data
    to the pipeline, use Write-Output (Write-Output.md)or implicit output.

    You can specify the color of text by using the `ForegroundColor` parameter, and you can specify the background color by using the `BackgroundColor` parameter. The
    Separator parameter lets you specify a string to use to separate displayed objects. The particular result depends on the program that is hosting PowerShell.

    &gt; [!NOTE] &gt; Starting in Windows PowerShell 5.0, `Write-Host` is a wrapper for `Write-Information` This allows &gt; you to use `Write-Host` to emit output to the
    information stream. This enables the capture or &gt; suppression of data written using `Write-Host` while preserving backwards compatibility. &gt; &gt; The
    `$InformationPreference` preference variable and `InformationAction` common parameter do not &gt; affect `Write-Host` messages. The exception to this rule is
    `-InformationAction Ignore`, which &gt; effectively suppresses `Write-Host` output. (see "Example 5")

RELATED LINKS
    Online Version: https://docs.microsoft.com/powershell/module/microsoft.powershell.utility/write-host?view=powershell-5.1&WT.mc_id=ps-gethelp
    Clear-Host
    Out-Host
    Write-Debug
    Write-Error
    Write-Output
    Write-Progress
    Write-Verbose
    Write-Warning

REMARKS
    To see the examples, type: "get-help Write-Host -examples".
    For more information, type: "get-help Write-Host -detailed".
    For technical information, type: "get-help Write-Host -full".
    For online help, type: "get-help Write-Host -online"
```

从备注信息，还可以知道，我们可以使用 -full 参数，查看完整地帮助，它除了包含默认地信息，还会包含 Parameters（参数），Example（示例）等信息；我们也可以添加 -examples 参数，只显示示例信息；-online 获取在线（最新） 的帮助信息。

除了这些，我们还可以通过 -ShowWindow 参数，在一个新的图形窗口来显示帮助信息；