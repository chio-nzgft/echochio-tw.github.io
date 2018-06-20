---
layout: post
title: 用 Mega 備份 資料
date: 2017-05-11
tags: Mega backup
---
先下載 linux 的 mega tools

https://megatools.megous.com/builds/megatools-1.9.98.tar.gz

放到 /usr/local/bin 下...... megacopy . megarm  ,  megarls ......

在家目錄建立 .megarc 檔案 ,  例如 /root/.megarc

內容如下 :

```
[Login]
Username = chio@test.com
Password = P@ssw0rd
```

執行 /usr/local/bin/megarls 看看是否可執行 , 如可那就正常了

寫個 /root/mega_backup.sh 定時備份檔 ...再放到 crontab 內

```
rm -rf /root/backuptime
touch /root/backuptime
tar -zcvf - /var/vmail |openssl des3 -salt -k password | dd of=/backup/vmail`date +"%Y%m%d"`.tgz.des3
rm -rf /backup/vmail`date -d '2 days ago' +%Y%m%d`.tgz.des3
/usr/local/bin/megarm /Root/vmail/vmail`date -d '2 days ago' +%Y%m%d`.tgz.des3
/usr/local/bin/megacopy -r /Root/vmail -l /backup
python /root/mega-info-mail.py
```

這備份檔會會刪除兩天前的檔案 ....請自行修改 .....2 days ago 那字眼 

備份檔的密碼是 password ....請自行修改 

還原檔案 (請自行修改 ...密碼 password  ) 測試用 tar zvtf -

```
dd if=/backup/vmail`date +"%Y%m%d"`.tgz.des3 |openssl des3 -d -k password|tar zvxf -
```

加了個 寄信到 gmail 的功能 ...用 python 寫的 .... /root/mega-info-mail.py

```
import smtplib
import subprocess
lsmega = subprocess.check_output(['/usr/local/bin/megals','-l'])
Subject = "Subject: "+ subprocess.check_output(['date'])+" Mega list"
fromaddr = 'testmonitor@gmail.com'
toaddrs  = 'testchio@gmail.com'
msg = "\r\n".join([
  "From: testmonitor@gmail.com",
  "To: testchio@gmail.com",
  Subject,
  "",
  lsmega
  ])
username = 'testmonitor@gmail.com'
password = 'P@ssw0rd'
server = smtplib.SMTP('smtp.gmail.com:587')
server.ehlo()
server.starttls()
server.login(username,password)
server.sendmail(fromaddr, toaddrs, msg)
server.quit()
```
