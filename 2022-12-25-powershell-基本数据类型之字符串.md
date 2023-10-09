---
title: PowerShell 基本数据类型之字符串
thumbnailImagePosition: left
thumbnailImage: https://img.freepik.com/premium-photo/concept-technology-internet-networking-hand-touching-digital-media-icon-with-show-display-tablet_2034-2453.jpg
coverImage: https://img.freepik.com/free-vector/abstract-medical-background-with-hexagonal-structure-shapes_1017-20046.jpg
metaAlignment: center
coverMeta: out
date: 2022-12-25
categories:
- PowerShell
tags:
- PowerShell
---

字符串由一个或多个字符组成，通常我们习惯加上引号，如 “Name”，‘Age’ 等等。

不同的数据类型，拥有不同的属性和方法，我们可以通过 Get-Member 来获取属性和方法

<!--more-->

```powershell
PS C:\Windows\system32> $logPath | Get-Member

   TypeName: System.String                      # 对象（变量）的数据类型为 string，Method 是其支持的方法，Property 是属性

Name             MemberType            Definition
----             ----------            ----------
Clone            Method                System.Object Clone(), System.Object ICloneable.Clone()
CompareTo        Method                int CompareTo(System.Object value), int CompareTo(string strB), int IComparab...
Contains         Method                bool Contains(string value)
CopyTo           Method                void CopyTo(int sourceIndex, char[] destination, int destinationIndex, int co...
EndsWith         Method                bool EndsWith(string value), bool EndsWith(string value, System.StringCompari...
Equals           Method                bool Equals(System.Object obj), bool Equals(string value), bool Equals(string...
GetEnumerator    Method                System.CharEnumerator GetEnumerator(), System.Collections.IEnumerator IEnumer...
GetHashCode      Method                int GetHashCode()
GetType          Method                type GetType()
GetTypeCode      Method                System.TypeCode GetTypeCode(), System.TypeCode IConvertible.GetTypeCode()
IndexOf          Method                int IndexOf(char value), int IndexOf(char value, int startIndex), int IndexOf...
IndexOfAny       Method                int IndexOfAny(char[] anyOf), int IndexOfAny(char[] anyOf, int startIndex), i...
Insert           Method                string Insert(int startIndex, string value)
IsNormalized     Method                bool IsNormalized(), bool IsNormalized(System.Text.NormalizationForm normaliz...
LastIndexOf      Method                int LastIndexOf(char value), int LastIndexOf(char value, int startIndex), int...
LastIndexOfAny   Method                int LastIndexOfAny(char[] anyOf), int LastIndexOfAny(char[] anyOf, int startI...
Normalize        Method                string Normalize(), string Normalize(System.Text.NormalizationForm normalizat...
PadLeft          Method                string PadLeft(int totalWidth), string PadLeft(int totalWidth, char paddingChar)
PadRight         Method                string PadRight(int totalWidth), string PadRight(int totalWidth, char padding...
Remove           Method                string Remove(int startIndex, int count), string Remove(int startIndex)
Replace          Method                string Replace(char oldChar, char newChar), string Replace(string oldValue, s...
Split            Method                string[] Split(Params char[] separator), string[] Split(char[] separator, int...
StartsWith       Method                bool StartsWith(string value), bool StartsWith(string value, System.StringCom...
Substring        Method                string Substring(int startIndex), string Substring(int startIndex, int length)
ToBoolean        Method                bool IConvertible.ToBoolean(System.IFormatProvider provider)
ToByte           Method                byte IConvertible.ToByte(System.IFormatProvider provider)
ToChar           Method                char IConvertible.ToChar(System.IFormatProvider provider)
ToCharArray      Method                char[] ToCharArray(), char[] ToCharArray(int startIndex, int length)
ToDateTime       Method                datetime IConvertible.ToDateTime(System.IFormatProvider provider)
ToDecimal        Method                decimal IConvertible.ToDecimal(System.IFormatProvider provider)
ToDouble         Method                double IConvertible.ToDouble(System.IFormatProvider provider)
ToInt16          Method                int16 IConvertible.ToInt16(System.IFormatProvider provider)
ToInt32          Method                int IConvertible.ToInt32(System.IFormatProvider provider)
ToInt64          Method                long IConvertible.ToInt64(System.IFormatProvider provider)
ToLower          Method                string ToLower(), string ToLower(cultureinfo culture)
ToLowerInvariant Method                string ToLowerInvariant()
ToSByte          Method                sbyte IConvertible.ToSByte(System.IFormatProvider provider)
ToSingle         Method                float IConvertible.ToSingle(System.IFormatProvider provider)
ToString         Method                string ToString(), string ToString(System.IFormatProvider provider), string I...
ToType           Method                System.Object IConvertible.ToType(type conversionType, System.IFormatProvider...
ToUInt16         Method                uint16 IConvertible.ToUInt16(System.IFormatProvider provider)
ToUInt32         Method                uint32 IConvertible.ToUInt32(System.IFormatProvider provider)
ToUInt64         Method                uint64 IConvertible.ToUInt64(System.IFormatProvider provider)
ToUpper          Method                string ToUpper(), string ToUpper(cultureinfo culture)
ToUpperInvariant Method                string ToUpperInvariant()
Trim             Method                string Trim(Params char[] trimChars), string Trim()
TrimEnd          Method                string TrimEnd(Params char[] trimChars)
TrimStart        Method                string TrimStart(Params char[] trimChars)
Chars            ParameterizedProperty char Chars(int index) {get;}
Length           Property              int Length {get;}

# 查看对象的数据类型
PS C:\Windows\system32> $logPath.GetType() 

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     String                                   System.Object

# 字符替换
PS C:\Windows\system32> $logPath = $logPath.Replace("C","E")
PS C:\Windows\system32> echo $logPath
E:\Logs\log.txt

# 字符串长度
PS C:\Windows\system32> $logPath = "E:\Logs\log.txt"
PS C:\Windows\system32> $logPath.Length
15

# 基于下标查找字符，第一个字符的下标为 0，以此递加；也可以将最后一个字符串的下标作为 -1，以此向前递减
PS C:\Windows\system32> $logPath[0]
E
PS C:\Windows\system32> $logPath[3]
L
PS C:\Windows\system32> $logPath[-1]
t

# 删除下标 0 到 8 的字符
PS C:\Windows\system32> $logPath.Remove(0,8)
log.txt

# 获取指定段的字符串
PS C:\Windows\system32> $a="Powershell"
PS C:\Windows\system32> $a.Substring(0,5)
Power

# 是否包含指定字符串
PS C:\Windows\system32> $logPath.Contains("log")
True
PS C:\Windows\system32> $logPath.Contains(".log")
False

# 替换输出，不会更改变量值
PS C:\Windows\system32> $logPath.Replace(".txt",".log")
E:\Logs\log.log
PS C:\Windows\system32> $logPath
E:\Logs\log.txt

# 切割字符串，括号内的为切割字符，默认是以空格作为切割字符
PS C:\Windows\system32> $logPath.Split("\")
E:
Logs
log.txt

PS C:\Windows\system32>  $logPath
E:\Logs\log.txt
```

在定义变量时，PowerShell 会根据变量的值，自定定义数据类型，但你也可以通过数据类型的关键字（如，string）来指定变量的数据类型

```powershell
PS C:\Windows\system32> $svr = "szsvr01"
PS C:\Windows\system32> $svr.GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     String                                   System.Object

PS C:\Windows\system32> $num = 123
PS C:\Windows\system32> $num.GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     Int32                                    System.ValueType

PS C:\Windows\system32> $num = "123"
PS C:\Windows\system32> $num.GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     String                                   System.Object
```