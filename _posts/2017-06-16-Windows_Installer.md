---
layout: post
title: Windows的安全模式時，系統將沒辨法啟動「Windows Installer」服務
date: 2017-06-16
tags: Windows Installer
---
不之為啥 win10 裝了 docekr 就登入很慢 ...等了半個多小時還不能用 .....XD

要如何移除 docker ....進入安全模式 ....

先在 CMD 啟動 Windows Installer 服務

```
REG ADD “HKLM\SYSTEM\CurrentControlSet\Control\SafeBoot\Minimal\MSIServer” /VE /T REG_SZ /F /D “Service” 
```

```
net start msiserver 
```

這樣就可移除了 .....XD

如是  Windows 安全模式(含網路功能)時

```
REG ADD “HKLM\SYSTEM\CurrentControlSet\Control\SafeBoot\Network\MSIServer” /VE /T REG_SZ /F /D “Service 
```
