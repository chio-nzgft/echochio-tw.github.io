---
layout: post
title: nginx 加網頁基本認證
date: 2018-06-25
tags: nginx
---

nginx 加網頁基本認證

我的OS 是 CentOS 裝 htpasswd

```
yum install -y httpd-tools
```

設定帳號 : echochio

```
[root@proxy nginx]# htpasswd -c /etc/nginx/.htpasswd echochio
New password:
Re-type new password:
Adding password for user echochio
[root@proxy nginx]#
``` 

編輯  /etc/nginx/conf.d/default.conf 加設定
放到適當的位置 ... 我的是 proxy , 所以放到特定網址內

```
#   auth_basic
    auth_basic "Private Property";
    auth_basic_user_file /etc/nginx/.htpasswd;
```

檢查 nginx 語法
```
[root@proxy nginx]#  nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

重啟 nginx

```
systemctl reload nginx
```
