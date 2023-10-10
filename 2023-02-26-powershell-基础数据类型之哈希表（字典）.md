---
title: PowerShell 基础数据类型之哈希表（字典）
thumbnailImagePosition: left
thumbnailImage: https://img.freepik.com/free-photo/application-programming-interface-hologram_23-2149092254.jpg
coverImage: https://img.freepik.com/premium-photo/metaverse-universe-concept-digital-neural-networkbusiness-man-hand-touching-introduction_102957-964.jpg
metaAlignment: center
coverMeta: out
date: 2023-02-26
categories:
- PowerShell
tags:
- PowerShell
---

哈希表也称为字典或关联阵列，是用于存储一个或多个键值对。在 PowerShell 中，哈希表被分为无序哈希表和有序哈希表。

<!--more-->

### 无序哈希表（Unordered hash table）
- 默认

```powershell
PS C:\Windows\system32> $mailList=@{"Tom"="Tom@abc.com";"Aaron"="Aaron@abc.com"}
PS C:\&gt; $mailList

Name                           Value
----                           -----
Aaron                          Aaron@abc.com
Tom                            Tom@abc.com

PS C:\Windows\system32> $mailList | Get-Member

   TypeName: System.Collections.Hashtable

Name              MemberType            Definition
----              ----------            ----------
Add               Method                void Add(System.Object key, System.Object value), void IDictionary.Add(Syste...
Clear             Method                void Clear(), void IDictionary.Clear()
Clone             Method                System.Object Clone(), System.Object ICloneable.Clone()
Contains          Method                bool Contains(System.Object key), bool IDictionary.Contains(System.Object key)
ContainsKey       Method                bool ContainsKey(System.Object key)
ContainsValue     Method                bool ContainsValue(System.Object value)
CopyTo            Method                void CopyTo(array array, int arrayIndex), void ICollection.CopyTo(array arra...
Equals            Method                bool Equals(System.Object obj)
GetEnumerator     Method                System.Collections.IDictionaryEnumerator GetEnumerator(), System.Collections...
GetHashCode       Method                int GetHashCode()
GetObjectData     Method                void GetObjectData(System.Runtime.Serialization.SerializationInfo info, Syst...
GetType           Method                type GetType()
OnDeserialization Method                void OnDeserialization(System.Object sender), void IDeserializationCallback....
Remove            Method                void Remove(System.Object key), void IDictionary.Remove(System.Object key)
ToString          Method                string ToString()
Item              ParameterizedProperty System.Object Item(System.Object key) {get;set;}
Count             Property              int Count {get;}
IsFixedSize       Property              bool IsFixedSize {get;}
IsReadOnly        Property              bool IsReadOnly {get;}
IsSynchronized    Property              bool IsSynchronized {get;}
Keys              Property              System.Collections.ICollection Keys {get;}
SyncRoot          Property              System.Object SyncRoot {get;}
Values            Property              System.Collections.ICollection Values {get;}

# 创建一个空的哈希表
PS C:\Windows\system32> $phoneList = @{}

# 获取所有键
PS C:\Windows\system32> $mailList.keys
Aaron
Tom

# 获取所有值
PS C:\Windows\system32> $mailList.Values
Aaron@abc.com
Tom@abc.com

# 查看指定键的值
PS C:\Windows\system32> $mailList.Tom
Tom@abc.com

# 更改指定键的值（有两种方式）
# 方法 1
PS C:\Windows\system32> $mailList["Tom"] = "Tom1@abc.com"
PS C:\Windows\system32> $mailList["Tom"]
Tom1@abc.com

# 方法 2
PS C:\Windows\system32> $mailList.Tom = "Tom@abc.com"
PS C:\Windows\system32> $mailList["tom"]
Tom@abc.com

# 添加键值对（也有两种方式）
PS C:\Windows\system32> $mailList["Roby"] = "Roby@abc.com"
PS C:\Windows\system32> $mailList

Name                           Value
----                           -----
Aaron                          Aaron@abc.com
Roby                           Roby@abc.com
Tom                            Tom@abc.com

PS C:\Windows\system32> $mailList.Add("Sam","Sam@abc.com")
PS C:\Windows\system32> $mailList

Name                           Value
----                           -----
Aaron                          Aaron@abc.com
Sam                            Sam@abc.com
Roby                           Roby@abc.com
Tom                            Tom@abc.com
```

### 有序哈希表（Ordered hash table）
- 关键字：Ordered
```powershell
PS C:\Windows\system32> $ip = [ordered]@{dcsvr01='192.168.1.100';dcsvr02='192.168.1.101';mailsvr01='192.168.1.150'}
PS C:\Windows\system32> $ip

Name                           Value
----                           -----
dcsvr01                        192.168.1.100
dcsvr02                        192.168.1.101
mailsvr01                      192.168.1.150
```