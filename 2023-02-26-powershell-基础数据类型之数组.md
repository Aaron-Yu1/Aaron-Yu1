---
title: PowerShell 基础数据类型之数组
thumbnailImagePosition: left
thumbnailImage: https://img.freepik.com/free-photo/programming-background-with-person-working-with-codes-computer_23-2150010125.jpg
coverImage: https://img.freepik.com/premium-photo/modern-workplace-with-laptop-computer-coffee-cup-office-supplies_67155-3379.jpg
metaAlignment: center
coverMeta: out
date: 2023-02-26
categories:
- PowerShell
tags:
- PowerShell
---

数组是一个用于存储一个或多个元素（也被称为 item）集合的数据结构。这些元素可以是同一个类型，也可以是不同的数据类型。

<!--more-->

定义一个数组
```powershell
PS C:\Windows\system32> $svrList = @("192.168.1.100","192.168.1.101")
PS C:\Windows\system32> $svrList
192.168.1.100
192.168.1.101

PS C:\Windows\system32> $svrList | Get-Member

   TypeName: System.String            # 数组支持的属性及方法取决于数组内元素的数据类型

Name             MemberType            Definition
----             ----------            ----------
Clone            Method                System.Object Clone(), System.Object ICloneable.Clone()
CompareTo        Method                int CompareTo(System.Object value), int CompareTo(string strB), int IComparable.Co...
Contains         Method                bool Contains(string value)
CopyTo           Method                void CopyTo(int sourceIndex, char[] destination, int destinationIndex, int count)
EndsWith         Method                bool EndsWith(string value), bool EndsWith(string value, System.StringComparison c...
Equals           Method                bool Equals(System.Object obj), bool Equals(string value), bool Equals(string valu...
GetEnumerator    Method                System.CharEnumerator GetEnumerator(), System.Collections.IEnumerator IEnumerable....
GetHashCode      Method                int GetHashCode()
... ...
... ...

PS C:\Windows\system32> $svrList.GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     Object[]                                 System.Array

# 同一个数组中，可以包含不同类似的数据
PS C:\Windows\system32> $array = 3, 5, "Tom"
PS C:\Windows\system32> $array
3
5
Tom

PS C:\Windows\system32> $array.GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     Object[]                                 System.Array

PS C:\Windows\system32> $array.Length               # 数组的长度
3

# 数组会为每个值按顺序分配一个索引（也被称为下标），从 0 开始
PS C:\Windows\system32> $svrList.IndexOf("192.168.1.100")
0

# 通过索引，我们可以查找到对应的值
PS C:\Windows\system32> $svrList[1]
192.168.1.101

# 指定索引范围内的元素
PS C:\Windows\system32> $svrList[0..1]
192.168.1.100
192.168.1.101

# -1 表示列表中的最后一个元素
PS C:\Windows\system32> $svrList[-1]
192.168.1.101

# 我们也可以通过下标的方式修改列表的值
PS C:\Windows\system32> $svrList[-1] = "192.168.1.150"
PS C:\Windows\system32> $svrList[-1]
192.168.1.150

# 另一种更改数组元素的方式，这种方式数组元素的下标不能为负数
PS C:\Windows\system32> $svrList.SetValue("192.168.1.101",1)      
PS C:\Windows\system32> $svrList
192.168.1.100
192.168.1.101

# 获取数组元素的长度
PS C:\Windows\system32> $svrList[1].Length
13

# 获取数组的元素个数，等于数组长度
PS C:\Windows\system32> $svrList.Count
2

# 添加元素到列表（实际上是合并两个列表，生成一个新的列表）
PS C:\Windows\system32> $svrList = $svrList + "192.168.1.150"  
PS C:\Windows\system32> $svrList
192.168.1.100
192.168.1.101
192.168.1.150
```

遍历数组中的元素
```powershell
PS C:\Windows\system32> $svrList.ForEach({$_})
192.168.1.100
192.168.1.101

PS C:\Windows\system32> $svrList.ForEach({ Test-NetConnection $_ })

ComputerName           : 192.168.1.100
RemoteAddress          : 192.168.1.100
InterfaceAlias         : Ethernet
SourceAddress          : 172.25.83.15
PingSucceeded          : True
PingReplyDetails (RTT) : 1 ms

ComputerName           : 192.168.1.101
RemoteAddress          : 192.168.1.101
InterfaceAlias         : Ethernet
SourceAddress          : 172.25.83.15
PingSucceeded          : True
PingReplyDetails (RTT) : 3 ms
```

清除数组中的值
```powershell
PS C:\Windows\system32> $svrList.Clear()
PS C:\Windows\system32> $svrList
```

我们还可以通过 [System.Collections.ArrayList] 来定义数组，这样定义的数组操作起来比较简单方便
```powershell
PS C:\Windows\system32> [System.Collections.ArrayList]$svr = "192.168.1.100","192.168.1.101"
PS C:\Windows\system32> $svr
192.168.1.100
192.168.1.101
PS C:\Windows\system32> $svr.GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     ArrayList                                System.Object

# 添加元素
PS C:\Windows\system32> $svr.Add("192.168.1.150")
2
PS C:\Windows\system32> $svr
192.168.1.100
192.168.1.101
192.168.1.150

# 删除元素
PS C:\Windows\system32> $svr.Remove("192.168.1.150")
PS C:\Windows\system32> $svr
192.168.1.100
192.168.1.101

# 插入元素
PS C:\Windows\system32> $svr.Insert(1,"192.168.1.150")
PS C:\Windows\system32> $svr
192.168.1.100
192.168.1.150
192.168.1.101

# 排序
PS C:\Windows\system32> $svr.Sort()
PS C:\Windows\system32> $svr
192.168.1.100
192.168.1.101
192.168.1.150
```