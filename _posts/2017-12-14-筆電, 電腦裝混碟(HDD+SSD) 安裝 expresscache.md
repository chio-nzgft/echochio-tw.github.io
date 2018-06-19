---
layout: post
title: 筆電, 電腦裝混碟(HDD+SSD) 安裝 expresscache
date: 2017-12-14
description: 筆電 電腦 混碟(HDD+SSD) 安裝 expresscache
tag: 筆電 電腦 混碟(HDD+SSD)
--- 

發現原廠的 混碟 沒作用 ....挖哩 .....

不知是不是舊版的 expresscache 在 win10 沒作用 .....XD

<img src="/images/posts/max-hdd/a1.png">

第二顆一看就知道是 SSD .....

下載  expresscache_x64 或 expresscache_x32 及 放入相關的機板 EC.LMF 到安裝目錄中

我放 lenovo 的版本到 git 內 

https://github.com/echochio-tw/expresscache

安裝後會放到

```
C:\Program Files\Condusiv Technologies\ExpressCache
```

在管理者權CMD限下可看 cache 

```
C:\WINDOWS\system32>eccmd -info
ExpressCache Command Version 1.3.118.0
Copyright?2010-2014 Condusiv Technologies.
Date Time: 12/14/2017 15:43:33:745 (SALES-NB #4)

EC Cache Info
==================================================

Mounted                   : Yes
Partition Size            : 22.36 GB
Reserved Size             : 3.00 MB
Volume Size               : 22.36 GB
Total Used Size           : 4.07 GB
Total Free Space          : 18.30 GB
Used Data Size            : 3.99 GB
Used Data Size on Disk    : 4.06 GB

Tiered Cache Stats
==================================================
Memory in use             : 960.00 MB
Blocks in use             : 7680
Read Percent              : 8.66%


Cache Stats
==================================================
Cache Volume Drive Number : 1
Total Read Count          : 62892
Total Read Size           : 2.69 GB
Total Cache Read Count    : 45293
Total Cache Read Size     : 1.96 GB
Total Write Count         : 8255
Total Write Size          : 299.09 MB
Total Cache Write Count   : 216
Total Cache Write Size    : 2.40 MB

Cache Read Percent        : 72.98%
Cache Write Percent       : 0.80%
```
