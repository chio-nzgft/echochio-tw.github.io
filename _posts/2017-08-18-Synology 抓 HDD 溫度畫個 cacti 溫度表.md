---
layout: post
title: Synology 抓 HDD 溫度畫個 cacti 溫度表
date: 2017-08-18
tags: Synology 溫度 cacti
---

開 ssh

我是改成 2222 post

ssh 進入後 再 sudo -i 到 root

看有沒有 smartctl

```
root@NAS~# which smartctl
/bin/smartctl
root@NAS~# /bin/smartctl
smartctl 6.5 (build date Jul  6 2017) [x86_64-linux-3.10.102] (local build)
Copyright (C) 2002-16, Bruce Allen, Christian Franke, www.smartmontools.org

ERROR: smartctl requires a device name as the final command-line argument.


Use smartctl -h to get a usage summary
```

沒有就去控制台裝

掃描 HDD & 看 HDD 與 取 溫度

```
root@NAS ~# smartctl --scan
/dev/hda -d ata # /dev/hda, ATA device
/dev/sda -d scsi # /dev/sda, SCSI device
/dev/sdb -d scsi # /dev/sdb, SCSI device

root@NAS ~#  smartctl -a /dev/hda -d ata | egrep "Model|Temp"
Model Family:     NAS HDD
Device Model:     ST4XXXXXXXX
190 Airflow_Temperature_Cel                                          0x0022   067   062   045    Old_age   Always       -       33 (Min/Max 31/33)
194 Temperature_Celsius                                              0x0022   033   040   000    Old_age   Always       -       33 (0 18 0 0 0)
```

改 /etc/sudoers 將 admin 改成不再問密碼

```
admin        ALL=NOPASSWD: ALL
```

 由遠端 ssh 進來 sudo .... 就可看到資料了
 
 網路上抄個 python 

```
#!/usr/bin/env python

import paramiko

hostname = '192.168.0.X'
port = 2222
username = 'User'
password = 'user_pass'
cmds = "sudo smartctl -a /dev/hda -d ata | awk ' /Serial Number:/ { printf \"%s: \", $NF }; /Temperature_Celsius/ { print $10 }; ' | sed -e 's/[-_]//g'

if __name__ == "__main__":
    paramiko.util.log_to_file('/dev/null')
    s = paramiko.SSHClient()
    s.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    s.connect(hostname, port, username, password)
    stdin, stdout, stderr = s.exec_command(cmds)
    print stdout.read()
    s.close()
```

 執行
 
```
#  python ssh.py
Z333F333: 33
```

 如何放到 cacti ...自己放把 ......
 
