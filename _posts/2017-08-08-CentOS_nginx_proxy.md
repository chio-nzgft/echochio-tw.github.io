---
layout: post
title: CentOS7 nginx proxy 定義 porxy port
date: 2017-08-08
tags:  CentOS nginx proxy
---

啟動 nginx 發現不可 listen 88 

 我的 config 是寫 
```
server {
    listen       88 ssl http2;
    server_name  test.com.tw;
    ssl_certificate    /etc/letsencrypt/live/test.com.tw/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/test.com.tw/privkey.pem;
    ssl_session_cache shared:SSL:50m;
........
```

啟動 nginx 出現

```
nginx: [emerg] bind() to 0.0.0.0:88 failed (13: Permission denied) selinux
```

加入 http_port_t ...出現 already defined

```
# yum install policycoreutils-python
# semanage port -a -t http_port_t  -p tcp 88
ValueError: Port tcp/88 already defined
```

要用  semanage port -m -t http_port_t -p tcp 88

```
# semanage port -m -t http_port_t -p tcp 88
# semanage port -l | grep http_port_t
http_port_t                    tcp      88, 80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
# systemctl restart nginx
```
