---
layout: post
title: ESXi 單機備份方案 ghettoVCB
date: 2017-08-11
tags:  ESXi ghettoVCB backup
---

ESXi 定時備份到 網路上的 NFS , 
例如 NAS https://www.echochio.nctu.me/2017/%E7%BE%A4%E8%BC%9D_NAS_FOR_ESXi.html

先打開 ESXI 伺服器的 SSH Service

<img src="/images/posts/Esxi_VCB/p1.png">

ESXi 6.X 要修改 可安裝 非原廠許可的 vib 檔 
先 ssh 進去修改
```
esxcli software acceptance set --level=CommunitySupported
```

下載 ghettoVCB 套件

https://github.com/lamw/ghettoVCB/raw/master/vghetto-ghettoVCB.vib

上傳到 datastore ...如   /vmfs/volumes/datastore1 

ssh 進去 ESXi 安裝 vghetto-ghettoVCB.vib

```
esxcli software vib install -v /vmfs/volumes/datastore1/vghetto-ghettoVCB.vib -f 
```

裝完後到  /opt/ghettovcb/bin/ 下會有兩個檔案:

```
ghettoVCB-restore.sh
ghettoVCB.sh
```

新增一個 ghettoVCB.conf 放到 datastore 如 /vmfs/volumes/NFS/backup
 ghettoVCB.conf 說明如下 :

```
VM_BACKUP_VOLUME=/vmfs/volumes/NFS/backup    #備份到本機的另一個儲存區
DISK_BACKUP_FORMAT=thin     #建議保留預設值,採用精簡佈建
VM_BACKUP_ROTATION_COUNT=2    #保留幾份 Guest OS 備份檔案
ENABLE_COMPRESSION=0    #備份時是否要壓縮 1啟動 , 0關閉
ENABLE_NON_PERSISTENT_NFS=0    #是否啟用自動掛載及卸載NFS儲存設備機制
UNMOUNT_NFS=0    #執行完就缷載NFS 資料夾 1啟動 , 0關閉
NFS_SERVER=192.168.0.X     #儲存設備 IP
NFS_MOUNT=/volumes1/NFS    #儲存設備(NAS) 分享的資料夾
NFS_VERSION=nfs4    #NFS版本，nfs / nfs4
NFS_LOCAL_NAME=NFS    #掛載到 /vmfs/volumes/ 下的名稱
NFS_VM_BACKUP_DIR=bckup     #esxi guest os 備份檔放到 backup 裡
SNAPSHOT_TIMEOUT=15   
EMAIL_LOG=1    #是否開啟 E-Mail 寄送 LOG。 1啟動 , 0關閉,如要寄送LOG必須再設定打開 SMTP 25PORT 及 FIREWALL
EMAIL_SERVER=msr.hinet.net
EMAIL_SERVER_PORT=25     
EMAIL_DELAY_INTERVAL=1     #是否延遲寄信
EMAIL_TO=example@gmail.com
EMAIL_ERRORS_TO=example@gmail.com
EMAIL_FROM=esxi@gmail.com
```

如果我的 ESXi 上的名為 web_12 的 guest 要備份

```
/opt/ghettovcb/bin/ghettoVCB.sh -g /vmfs/volumes/NFS/backup/ghettoVCB.conf -m web_12
```

如寫成列表檔 

```
#cat /vmfs/volumes/NFS/backup/vmlist
mail_proxy
web_12
```

設定 firewall.sh 讓 smtp 可以寄信

```
# cat /vmfs/volumes/NFS/backup/firewall.sh
cp /vmfs/volumes/NFS/backup/smtp.xml /etc/vmware/firewall/
esxcli network firewall refresh
```

其中 smtp.xml 內容 : (id 要看 /etc/vmware/firewall/service.xml ...不能重複 我設200 不太會重複) 

```
<ConfigRoot>
  <service id='0200'>
    <id>SMTP client</id>
   <rule id='0000'>
     <direction>outbound</direction>
     <protocol>tcp</protocol>
     <porttype>dst</porttype>
     <port>25</port>
   </rule>
       <enabled>true</enabled>
     <required>true</required>
   </service>
</ConfigRoot>
```

執行 /vmfs/volumes/NFS/backup/firewall.sh 後
利用 vSphere Client 確認 SMTP 25 Port 是否已開放:

<img src="/images/posts/Esxi_VCB/p2.png">

設定 /vmfs/volumes/NFS/backup/crondtab.sh
之後執行讓排程正常運作

```
 #!/bin/sh
/bin/kill $(cat /var/run/crond.pid)
echo "30 1 * * 2,4,6 /opt/ghettovcb/bin/ghettoVCB.sh -g /vmfs/volumes/NFS/backup/ghettoVCB.conf -f /vmfs/volumes/NFS/backup/vmlist -l /tmp/vmbackup_$\(date +%F_%H-%M).log > /dev/null" >> /var/spool/cron/crontabs/root
/usr/lib/vmware/busybox/bin/busybox crond
```

執行 /vmfs/volumes/NFS/backup/crondtab.sh 後看 /var/spool/cron/crontabs/root 有沒有 排程

```
#min hour day mon dow command
1    1    *   *   *   /sbin/tmpwatch.py
1    *    *   *   *   /sbin/auto-backup.sh
0    *    *   *   *   /usr/lib/vmware/vmksummary/log-heartbeat.py
*/5  *    *   *   *   /sbin/hostd-probe ++group=host/vim/vmvisor/hostd-probe
*/2 * * * * /usr/lib/vmware/vsan/bin/vsanObserver.sh
30 1 * * 2,4,6 /opt/ghettovcb/bin/ghettoVCB.sh -g /vmfs/volumes/NFS/backup/ghettoVCB.conf -f /vmfs/volumes/NFS/backup/vmlist -l /tmp/vmbackup_$(date +%F_%H-%M).log > /dev/null
```

將  crondtab.sh 與 firewall.sh 放入  /etc/rc.local.d/local.sh 讓 ESXi 啟動時執行

```
/vmfs/volumes/NFS/backup/crondtab.sh
/vmfs/volumes/NFS/backup/firewall.sh
```
