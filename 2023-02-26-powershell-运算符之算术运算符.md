---
title: Powershell 运算符之算术运算符
thumbnailImagePosition: left
thumbnailImage: https://img.freepik.com/free-photo/numerology-concept-composition_23-2150169791.jpg
coverImage: https://img.freepik.com/premium-photo/thinking-ai-humanoid-robot-analyzing-screen-mathematics-formula-science_31965-33974.jpg
metaAlignment: center
coverMeta: out
date: 2023-02-26
categories:
- PowerShell
tags:
- PowerShell
---

在 Powershell 中，我们可以通过算术运算符对数值进行数值计算。

<!--more-->

PowerShell 支持：
- +（加法）
- -（减法）
- \*（乘法）
- /（除法）
- %（取余）

### 加法

```powershell
# 两个数相加
PS C:\Windows\system32> 5 + 3
8

# 两个字符串相加
PS C:\Windows\system32> "Aaron" + "Yu"
aaronyu

# 当两个不同类型数据进行算术运算时，Powershell 会自动对后者进行数据类型转换
PS C:\Windows\system32> 3 + "4"
7
PS C:\Windows\system32> "4" + 3
43

# 两个数组相加
PS C:\Windows\system32> $a = @("192.168.1.100")
PS C:\Windows\system32> $b = @("192.168.1.101")
PS C:\Windows\system32> $c = $a + $b
PS C:\Windows\system32> $c
192.168.1.100
192.168.1.101
PS C:\Windows\system32> $c.GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     Object[]                                 System.Array

# 两个哈希表相加
PS C:\Windows\system32> $a = @{dcsvr01="192.168.1.100";dcsvr02="192.168.1.101"}
PS C:\Windows\system32> $b = @{mailsvr01="192.168.1.150"}
PS C:\Windows\system32> $c = $a + $b
PS C:\Windows\system32> $c

Name                           Value
----                           -----
dcsvr01                        192.168.1.100
mailsvr01                      192.168.1.150
dcsvr02                        192.168.1.101

PS C:\Windows\system32> $c.GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     Hashtable                                System.Object
```

### 减法
```powershell
PS C:\Windows\system32> 4 -2
2
PS C:\Windows\system32> "4" - 2
2
PS C:\Windows\system32> 4 - "2"
2
```

### 乘法

```powershell
# 两个整数相乘
PS C:\Windows\system32> 2 * 3
6

# 通过乘法复制字符串
PS C:\Windows\system32> "Powershell" * 3
PowershellPowershellPowershell

# 通过乘法复制列表元素
PS C:\Windows\system32> $a = 1,2,3
PS C:\Windows\system32> $b = $a * 3
PS C:\Windows\system32> $b
1
2
3
1
2
3
1
2
3
```

### 除法
```powershell
PS C:\Windows\system32> $a = 15/3
PS C:\Windows\system32> $a.GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     Int32                                    System.ValueType
PS C:\Windows\system32> $b = 7/2
PS C:\Windows\system32> $b
3.5
PS C:\Windows\system32> $b.GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     Double                                   System.ValueType

# 通过 [int] 实现四舍五入
PS C:\Windows\system32> [int]$c = 7/2
PS C:\Windows\system32> $c
4
PS C:\Windows\system32> $c.GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     Int32                                    System.ValueType
```

### 取余
```powershell
PS C:\Windows\system32> 14 % 6
2
```

### [Math]
在 PowerShell 中没有幂运算符，如果想在 PowerShell 中进行幂运算，我们需要借助 [Math] Class(.NET Class)。通过 [Math] 我们可以进行更加复杂的数学计算。

```powershell
PS C:\Windows\system32> # 幂运算
PS C:\Windows\system32> [math]::Pow(2,4)
16

# 四舍五入
PS C:\Windows\system32> [math]::Round(3.1415926)
3
PS C:\Windows\system32> [math]::Round(3.1415926,2)   # 后面 2 表示保留 2 位小数
3.14

# 平方根
PS C:\Windows\system32> [math]::Sqrt(81)
9

# 支持的方法（Method）
PS C:\Windows\system32> [math].GetMethods() | select Name -Unique

Name
----
Acos
Asin
Atan
Atan2
Ceiling
Cos
Cosh
Floor
Sin
Tan
Sinh
Tanh
Round
Truncate
Sqrt
Log
Log10
Exp
Pow
IEEERemainder
Abs
Max
Min
Sign
BigMul
DivRem
ToString
Equals
GetHashCode
GetType
```

更多信息，请参考 [Math Class](https://learn.microsoft.com/en-us/dotnet/api/system.math?view=net-7.0) 方法

算术运算符优先级由高到底
- ()
- -（负号，一元运算符）
- *, /, %
- +, -

```powershell
# 优先计算括号内的，然后从左往右计算
PS C:\Windows\system32> (3 + 6) / 3 * 4
12
PS C:\Windows\system32> 3 + -2
1

# 先计算除，然后计算乘，最后与前面的 3 相加
PS C:\Windows\system32> 3 + 6 / 3 * 4    
11
```