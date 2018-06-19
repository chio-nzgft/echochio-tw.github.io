---
layout: post
title: ESXi 單機 ghettoVCB 的還原
date: 2017-08-12
tags:  ESXi ghettoVCB restore
---

編輯 re-vmlist (我放到 /vmfs/volumes/NFS/backup/re-vmlist)
```
# DISK_FORMATS
# 1 = zeroedthick
# 2 = 2gbsparse
# 3 = thin
# 4 = eagerzeroedthick
"/vmfs/volumes/NFS/backup/web_12/web_12-2017-08-10_07-13-34;/vmfs/volumes/datastore2;3"
```

說明如下: 
```
# DISK_FORMATS
# 1 = zeroedthick  (一次給足全部的硬碟大小，需要時才初使化未使用的空間)
# 2 = 2gbsparse  (將硬碟分割成多個 2G 的硬碟)
# 3 = thin  (隨著使用量而增加硬碟大小，達到設定上限時就不會在增加)
# 4 = eagerzeroedthick  (一次給足全部的硬碟大小(刪除所有的資料)，已初始化可直接使用)
"還原來源 目錄 ; 還原的目的 store ; 磁碟存放方式"
```

Dryrun(模擬執行):
```
/opt/ghettovcb/bin/ghettoVCB-restore.sh -c /vmfs/volumes/NFS/backup/re-vmlist -d 1
```

Debug(模擬執行):
```
/opt/ghettovcb/bin/ghettoVCB-restore.sh -c /vmfs/volumes/NFS/backup/re-vmlist -d 2
```

正式還原
```
/opt/ghettovcb/bin/ghettoVCB-restore.sh -c  /vmfs/volumes/NFS/backup/re-vmlist -l /tmp/vmrestore.log
```
