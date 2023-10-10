---
title: PowerShell 基础数据类型之时间
thumbnailImagePosition: left
thumbnailImage: https://img.freepik.com/free-photo/lipstick-makeup-brush-alarm-clock-eyeglasses-laptop-spiral-notepad-peach-background_23-2148178644.jpg
coverImage: https://img.freepik.com/free-vector/geometric-astrological-symbols-tarot-card_53876-82179.jpg
metaAlignment: center
coverMeta: out
date: 2023-01-23
categories:
- PowerShell
tags:
- PowerShell
---

时间类型是 powershell 中另外一个重要的数据类型。我们常常需要基于时间，获取或标记数据。

<!--more-->

```powershell
PS C:\Windows\system32> Get-Date 
PS C:\Windows\system32> $today = Get-Date
PS C:\Windows\system32> $today.GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     DateTime                                 System.ValueType

# 时间数据类型支持的方法
PS C:\Windows\system32>  $today | Get-Member

   TypeName: System.DateTime

Name                 MemberType     Definition
----                 ----------     ----------
Add                  Method         datetime Add(timespan value)
AddDays              Method         datetime AddDays(double value)
AddHours             Method         datetime AddHours(double value)
AddMilliseconds      Method         datetime AddMilliseconds(double value)
AddMinutes           Method         datetime AddMinutes(double value)
AddMonths            Method         datetime AddMonths(int months)
AddSeconds           Method         datetime AddSeconds(double value)
AddTicks             Method         datetime AddTicks(long value)
AddYears             Method         datetime AddYears(int value)
CompareTo            Method         int CompareTo(System.Object value), int CompareTo(datetime value), int IComparab...
Equals               Method         bool Equals(System.Object value), bool Equals(datetime value), bool IEquatable[d...
GetDateTimeFormats   Method         string[] GetDateTimeFormats(), string[] GetDateTimeFormats(System.IFormatProvide...
GetHashCode          Method         int GetHashCode()
GetObjectData        Method         void ISerializable.GetObjectData(System.Runtime.Serialization.SerializationInfo ...
GetType              Method         type GetType()
GetTypeCode          Method         System.TypeCode GetTypeCode(), System.TypeCode IConvertible.GetTypeCode()
IsDaylightSavingTime Method         bool IsDaylightSavingTime()
Subtract             Method         timespan Subtract(datetime value), datetime Subtract(timespan value)
ToBinary             Method         long ToBinary()
ToBoolean            Method         bool IConvertible.ToBoolean(System.IFormatProvider provider)
ToByte               Method         byte IConvertible.ToByte(System.IFormatProvider provider)
ToChar               Method         char IConvertible.ToChar(System.IFormatProvider provider)
ToDateTime           Method         datetime IConvertible.ToDateTime(System.IFormatProvider provider)
ToDecimal            Method         decimal IConvertible.ToDecimal(System.IFormatProvider provider)
ToDouble             Method         double IConvertible.ToDouble(System.IFormatProvider provider)
ToFileTime           Method         long ToFileTime()
ToFileTimeUtc        Method         long ToFileTimeUtc()
ToInt16              Method         int16 IConvertible.ToInt16(System.IFormatProvider provider)
ToInt32              Method         int IConvertible.ToInt32(System.IFormatProvider provider)
ToInt64              Method         long IConvertible.ToInt64(System.IFormatProvider provider)
ToLocalTime          Method         datetime ToLocalTime()
ToLongDateString     Method         string ToLongDateString()
ToLongTimeString     Method         string ToLongTimeString()
ToOADate             Method         double ToOADate()
ToSByte              Method         sbyte IConvertible.ToSByte(System.IFormatProvider provider)
ToShortDateString    Method         string ToShortDateString()
ToShortTimeString    Method         string ToShortTimeString()
ToSingle             Method         float IConvertible.ToSingle(System.IFormatProvider provider)
ToString             Method         string ToString(), string ToString(string format), string ToString(System.IForma...
ToType               Method         System.Object IConvertible.ToType(type conversionType, System.IFormatProvider pr...
ToUInt16             Method         uint16 IConvertible.ToUInt16(System.IFormatProvider provider)
ToUInt32             Method         uint32 IConvertible.ToUInt32(System.IFormatProvider provider)
ToUInt64             Method         uint64 IConvertible.ToUInt64(System.IFormatProvider provider)
ToUniversalTime      Method         datetime ToUniversalTime()
DisplayHint          NoteProperty   DisplayHintType DisplayHint=DateTime
Date                 Property       datetime Date {get;}
Day                  Property       int Day {get;}
DayOfWeek            Property       System.DayOfWeek DayOfWeek {get;}
DayOfYear            Property       int DayOfYear {get;}
Hour                 Property       int Hour {get;}
Kind                 Property       System.DateTimeKind Kind {get;}
Millisecond          Property       int Millisecond {get;}
Minute               Property       int Minute {get;}
Month                Property       int Month {get;}
Second               Property       int Second {get;}
Ticks                Property       long Ticks {get;}
TimeOfDay            Property       timespan TimeOfDay {get;}
Year                 Property       int Year {get;}
DateTime             ScriptProperty System.Object DateTime {get=if ((& { Set-StrictMode -Version 1; $this.DisplayHin...

# 5 天后的时间
PS C:\Windows\system32> $today.AddDays(5)

Tuesday, December 6, 2022 3:16:20 PM

# 5 天前的时间
PS C:\Windows\system32>  $today.AddDays(-5)

Saturday, November 26, 2022 3:16:20 PM

# 5 小时后
PS C:\Windows\system32> $today.AddHours(5)

Thursday, December 1, 2022 8:16:20 PM

# 5 月后
PS C:\Windows\system32> $today.AddMonths(5)

Monday, May 1, 2023 3:16:20 PM

# 5 年后
PS C:\Windows\system32> $today.AddYears(3)

Monday, December 1, 2025 3:16:20 PM

# 今天的日期（天，也就是多少号）
PS C:\Windows\system32> $today.Day
1

# 今天的年份
PS C:\Windows\system32> $today.Year
2022

# 今天月份
PS C:\Windows\system32> $today.Month
12

# 今天是周几
PS C:\Windows\system32> $today.DayOfWeek
Thursday

# 组合时间成文件名
PS C:\Windows\system32> $logFile = [string]$today.Year + "-" + $today.Month + "-" + $today.Day + "-" + $today.Hour + "-" + $today.Minute + ".txt"
PS C:\Windows\system32> $logFile
2022-12-1-15-16.txt
```