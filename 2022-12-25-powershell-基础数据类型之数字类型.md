---
thumbnailImagePosition: left
thumbnailImage: https://img.freepik.com/free-vector/business-doodles-blackboard_1284-44664.jpg
coverImage: https://img.freepik.com/premium-photo/cube-computing-technology-cloud-data-internet-security_505353-643.jpg
metaAlignment: center
coverMeta: out
date: 2022-12-25
categories:
- PowerShell
tags:
- PowerShell
---

同样在 Powershell 中，数字类型也被划分了：

  * 整数
  * 浮点数

<!--more-->

#### 整数

在 PowerShell 中，整数和其他语言中的定义没有区别，就是我们数学上定义的整数，如 123, 234, 22 等等。

```powershell
PS C:\Windows\system32> $num = 123
PS C:\Windows\system32> $num | Get-Member

   TypeName: System.Int32

Name        MemberType Definition
----        ---------- ----------
CompareTo   Method     int CompareTo(System.Object value), int CompareTo(int value), int IComparable.CompareTo(Syste...
Equals      Method     bool Equals(System.Object obj), bool Equals(int obj), bool IEquatable[int].Equals(int other)
GetHashCode Method     int GetHashCode()
GetType     Method     type GetType()
GetTypeCode Method     System.TypeCode GetTypeCode(), System.TypeCode IConvertible.GetTypeCode()
ToBoolean   Method     bool IConvertible.ToBoolean(System.IFormatProvider provider)
ToByte      Method     byte IConvertible.ToByte(System.IFormatProvider provider)
ToChar      Method     char IConvertible.ToChar(System.IFormatProvider provider)
ToDateTime  Method     datetime IConvertible.ToDateTime(System.IFormatProvider provider)
ToDecimal   Method     decimal IConvertible.ToDecimal(System.IFormatProvider provider)
ToDouble    Method     double IConvertible.ToDouble(System.IFormatProvider provider)
ToInt16     Method     int16 IConvertible.ToInt16(System.IFormatProvider provider)
ToInt32     Method     int IConvertible.ToInt32(System.IFormatProvider provider)
ToInt64     Method     long IConvertible.ToInt64(System.IFormatProvider provider)
ToSByte     Method     sbyte IConvertible.ToSByte(System.IFormatProvider provider)
ToSingle    Method     float IConvertible.ToSingle(System.IFormatProvider provider)
ToString    Method     string ToString(), string ToString(string format), string ToString(System.IFormatProvider pro...
ToType      Method     System.Object IConvertible.ToType(type conversionType, System.IFormatProvider provider)
ToUInt16    Method     uint16 IConvertible.ToUInt16(System.IFormatProvider provider)
ToUInt32    Method     uint32 IConvertible.ToUInt32(System.IFormatProvider provider)
ToUInt64    Method     uint64 IConvertible.ToUInt64(System.IFormatProvider provider)


# 转换成字符串
PS C:\Windows\system32> $a = $num.ToString()
PS C:\&gt; $a.GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     String                                   System.Object
```

#### 浮点数

浮点数，即我们常说的带小数部分的数，如，3.14 等等；

```powershell
PS C:\Windows\system32> $num = 3.14
PS C:\Windows\system32> $num.GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     Double                                   System.ValueType

PS C:\Windows\system32> $num | Get-Member

   TypeName: System.Double

Name        MemberType Definition
----        ---------- ----------
CompareTo   Method     int CompareTo(System.Object value), int CompareTo(double value), int IComparable.CompareTo(Sy...
Equals      Method     bool Equals(System.Object obj), bool Equals(double obj), bool IEquatable[double].Equals(doubl...
GetHashCode Method     int GetHashCode()
GetType     Method     type GetType()
GetTypeCode Method     System.TypeCode GetTypeCode(), System.TypeCode IConvertible.GetTypeCode()
ToBoolean   Method     bool IConvertible.ToBoolean(System.IFormatProvider provider)
ToByte      Method     byte IConvertible.ToByte(System.IFormatProvider provider)
ToChar      Method     char IConvertible.ToChar(System.IFormatProvider provider)
ToDateTime  Method     datetime IConvertible.ToDateTime(System.IFormatProvider provider)
ToDecimal   Method     decimal IConvertible.ToDecimal(System.IFormatProvider provider)
ToDouble    Method     double IConvertible.ToDouble(System.IFormatProvider provider)
ToInt16     Method     int16 IConvertible.ToInt16(System.IFormatProvider provider)
ToInt32     Method     int IConvertible.ToInt32(System.IFormatProvider provider)
ToInt64     Method     long IConvertible.ToInt64(System.IFormatProvider provider)
ToSByte     Method     sbyte IConvertible.ToSByte(System.IFormatProvider provider)
ToSingle    Method     float IConvertible.ToSingle(System.IFormatProvider provider)
ToString    Method     string ToString(), string ToString(string format), string ToString(System.IFormatProvider pro...
ToType      Method     System.Object IConvertible.ToType(type conversionType, System.IFormatProvider provider)
ToUInt16    Method     uint16 IConvertible.ToUInt16(System.IFormatProvider provider)
ToUInt32    Method     uint32 IConvertible.ToUInt32(System.IFormatProvider provider)
ToUInt64    Method     uint64 IConvertible.ToUInt64(System.IFormatProvider provider)

# 转换为整数
PS C:\Windows\system32> $num.ToInt32($null)
3

# 我们也可以直接使用 [int] 进行转换
PS C:\Windows\system32> [int]$num
3
```