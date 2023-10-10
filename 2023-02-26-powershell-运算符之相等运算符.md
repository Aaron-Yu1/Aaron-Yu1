---
title: Powershell 运算符之相等运算符（等式）
thumbnailImagePosition: left
thumbnailImage: https://img.freepik.com/premium-photo/cloud-technology-concept-with-digital-tablet-human-hand-virtual-projection-with-different-cloud-service-signs-security-loupe-computer-data-blurry-office-background_670147-20058.jpg
coverImage: https://img.freepik.com/premium-photo/cloud-computing-technology-internet-converging-point-circuit-with-abstract-blue-background-cloud-service-cloud-storage-concept-3d-illustration_505353-311.jpg
metaAlignment: center
coverMeta: out
date: 2023-02-26
categories:
- PowerShell
tags:
- PowerShell
---

相等运算符用于判断两边的值是否相等，或大于，或小于。

<!--more-->

PowerShell 中相等运算符有：
- -eq: 等于（euqal to）
- -ne: 不等于（not euqual to）
- -gt: 大于（greater than）
- -ge: 大于或等于（greater than or euqal to）
- -lt: 小于（less than）
- -le: 小于或等于（less than or equal to）

可以在所有运算符前面加一个 c，如，-ceq，cne，等等，表示区分大小写
```powershell
# 等于
PS C:\Windows\system32> 10 -eq 5
False

# -eq 默认是不区分大小写的
PS C:\Windows\system32> "Powershell" -eq "powershell"
True

# -ceq 即 case-sensitive equal，区分大小写的相等比较
PS C:\Windows\system32> "Powershell" -ceq "powershell"
False
```

在比较时，未定义（声明）的变量（null）和空值得变量是不一样的。
```powershell
# $x 未定义
PS C:\Windows\system32> $x -eq $null          
True
PS C:\Windows\system32> $a = ''
PS C:\Windows\system32> $a -eq $null
False
PS C:\Windows\system32> $b = $null
PS C:\Windows\system32> $b -eq $null
False
```

相等运算符可以比较不同类型的对象。 但是需要注意，它是将比较右侧的值的类型转换为左侧值的类型，然后进行比较
```powershell
PS C:\Windows\system32> 1 -eq "1.0"
True
PS C:\Windows\system32> "1.0" -eq 1
False
```

-gt、-ge、-lt 和 -le
```powershell
# 不等于
PS C:\Windows\system32> $b -ne $null
True

# 大于
PS C:\Windows\system32> 5 -gt 3
True

# 按照 ASCII 码表 a 在 z 前，对应的数字 a 小于 z
PS C:\Windows\system32> "a" -gt "z"
False

# 大于或等于
PS C:\Windows\system32> 8 -ge 8
True
PS C:\Windows\system32> 8 -ge 4
True

# 小于 
PS C:\Windows\system32> 5 -lt 8
True

# 小于或等于
PS C:\Windows\system32> 6 -le 8
True
PS C:\Windows\system32> 8 -le 8
True
```

### 时间比较

我们还可以通过相等运算符，对两个时间值进行比较
```powershell
PS C:\Windows\system32> '2001-11-12' -lt '2020-08-01'
True
PS C:\Windows\system32> '2022-12-6' -gt '2021-12-04'
True
```