---
layout: post
title: python ssh 到 Esxi 查看磁碟狀況
date: 2018-07-07
tags: Esxi python storcli lsi
---

以前發生 MegaRAID 的硬碟掛了沒通知 mail ....

改進一下每天檢查看硬碟狀況

要安裝 storcli 在 Esxi 上 , 改天來寫

寫個 python 放入排程每天定時 ssh 到 去檢查

```
#!/usr/bin/env python

#load paramiko for ssh 
import sys, paramiko
hostname = '192.168.0.X'
username = 'root'
password = 'echochio'
command = '/opt/lsi/storcli/storcli /c0/dall show all |grep 252'
port = 22

try:
    client = paramiko.SSHClient()
    client.load_system_host_keys()
    client.set_missing_host_key_policy(paramiko.WarningPolicy)
    client.connect(hostname, port=port, username=username, password=password)
    stdin, stdout, stderr = client.exec_command(command)
    datainp = stdout.read(),

finally:
    client.close()

# datainp type tuple ...
# use join tuple as string
# replace \n to <br>
data = ''.join(datainp).replace('\n', '<br>')
import smtplib
from email.mime.text import MIMEText
from email.header import Header
smtpserver='192.168.0.X'
user='echochio'
password='echochio'
sender='echochio@echochio.nctu.me'
receiver='echochio@echochio.nctu.me'
#count Onln string
info=data.count("Onln")
# change info to integer for check is 4 ?
if int(info) == 4:
        subject='Python mail esxi disk OK log'
else:
        subject='Error !!  esxi disk !!! '
msg=MIMEText(data,'html','utf-8')
msg['Subject']=Header(subject,'utf-8')
smtp=smtplib.SMTP()
smtp.connect(smtpserver)
smtp.login(user, password)
smtp.sendmail(sender, receiver, msg.as_string())
smtp.quit()
print("sent mail !!")
```
