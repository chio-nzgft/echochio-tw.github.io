---
layout: post
title: windows 停用 user 的排程
date: 2017-06-30
tags: windows
---

話說以前做過 用 web 介面 , 給NOC 操作廠商遠端登入帳號建立及啟用, 停用等等 ....
當然有配合遠端登入錄影 ,
那時測過 RecordTS 及 ObserveIT records , 最後選ObserveIT records 

好了不廢話 .....
那就 telnet  ( 最後裝 openssh .....)
到 client (Windows XP) ....為啥用 XP ...因為 XP 可多人遠端桌面 ....哈哈 ....

下載 Universal Termsrv Patch
執行 xp.reg
UniversalTermsrvPatch-x86.exe (若作業系統為 64 位元版本則執行 x64.exe)
按「破解」，重開機

聽說後來 win7 也可多人了找 ... UniversalTermsrvPatch_20090425
好像是 SP1 可以 ....SP2 就不行了 ....XD
win7 SP2 後來的請用 

https://www.echochio.nctu.me/2018/06/win7-RDP/

那時用 AT .....現在要用 SCHTASKS

那就紀錄一下方法 

建立停用帳號排程
```
SCHTASKS /Create /ST 18:30:00 /SC ONCE /SD 2017/06/30 /TN stopactive-once-1830 /TR "net user eltsai /active:no"
SCHTASKS /Create /ST 18:31:00 /SC ONCE /SD 2017/06/30 /TN stopactive-once-1831 /TR "net user miyachiang /active:no"
```

看排程
```
schtasks | find /i "stopactive"
```

刪除排程
```
SCHTASKS /Delete /TN "stopactive-once-1830"
```
 
