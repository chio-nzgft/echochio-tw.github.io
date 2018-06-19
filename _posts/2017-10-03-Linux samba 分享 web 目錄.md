---
layout: post
title: Linux samba 分享 web 目錄
date: 2017-10-03
tags: Linux samba 分享
---

 發現上圖寫個 web 後台有些麻煩  , scp 又有些麻煩最簡單就是
 
網芳了 samba 的 /etc/samba/smb.conf 設定如下 :

----------
```
[global]
        workgroup = SAMBA
        security = user

        passdb backend = tdbsam

[web_php]
    path = /var/www/html/prog
    force user = apache
    browseable = No
    read only = No
    inherit acls = Yes

[web_picture]
    path = /var/www/html/image
    force user = apache
    browseable = No
    read only = No
    inherit acls = Yes
```
----------
加入 user 帳號 apache (看你的 web 帳號 ...我的是 apache )
----------
```
smbpasswd -a  apache 
```
----------
