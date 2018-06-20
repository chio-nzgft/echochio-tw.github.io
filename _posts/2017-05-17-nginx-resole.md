---
layout: post
title: 用 nginx resolve proxy 將自己的 http/1.1 變成 http/2.0
date: 2017-05-17
tags: nginx proxy http/2.0
---
將原本的服務的 port 例如 443 改成 4443

再由 nginx proxy 去瀏覽 4443 然而對外的是 nginx 443 port , nginx 用 http/2.0

這樣 不管後台是啥都變成  http/2.0 了

centos 裝 nginx

編輯 /etc/yum.repos.d/nginx.repo

```
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1
```

安裝 ngnix

```
yum install nginx
```

編輯 /etc/nginx/conf.d/default.conf (主機是 http://ssl.demo.com  變 443 )

```
server {
    listen       443 ssl http2;
    server_name  eip.enlicom.com;
    ssl_certificate    /etc/letsencrypt/live/ssl.demo.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/ssl.demo.com/privkey.pem;
    ssl_session_cache shared:SSL:50m;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:ECDHE-RSA-DES-CBC3-SHA:ECDHE-ECDSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA;
    ssl_prefer_server_ciphers on;
    ssl_dhparam /etc/pki/tls/dhparams.pem;
    resolver 8.8.8.8;
     location / {
        add_header X-Proxy-Cache    $upstream_cache_status;
        proxy_pass                  http://ssl.demo.com;
        proxy_set_header        Host $host;
        proxy_set_header        X-Real-IP $remote_addr;
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header        X-Forwarded-Proto $scheme;
        proxy_read_timeout  90;
        proxy_redirect      http://ssl.demo.com https://ssl.demo.com;
        sub_filter_types text/xml text/css *;
        sub_filter 'http://'  'https://';
        sub_filter_once off;
    }
     location ~ /.well-known {
        allow all;
    }
}
```

其中 ....
更改下面文件字串 ....最後那是 "星號" 是指全部 .....

```
 sub_filter_types text/xml text/css *;
```

將 http://ssl.demo.com 換成 https://ssl.demo.com
(要看需求 ....或許用 http:// 換成 https:// 先測試比較好)
這與 linux sed 功能很像

```
sub_filter 'http://ssl.demo.com'  'https://ssl.demo.com';
```

只換第一個找到的 字串更換

```
sub_filter_once off;
```

這段是給我的 certificate renew 用的 .....

```
 location ~ /.well-known {
        allow all;
}
```

還需注意的是 後端傳來的文件不能是壓縮過的例如 gzip 文件 , 壓縮過的 gzip 文件 sub_filter 不能處理
處理方式加 

```
proxy_set_header Accept-Encoding "";
```

原本是 :
```
# /usr/local/bin/curl --http2 -I --insecure https://ssl.demo.com
HTTP/1.1 200 OK
Date: Wed, 17 May 2017 08:25:07 GMT
Server: Apache/2.2.15 (CentOS)
Last-Modified: Wed, 17 May 2017 07:32:24 GMT
ETag: "2c0b72-9-54fb34a123200"
Accept-Ranges: bytes
Content-Length: 9
Connection: close
Content-Type: text/html; charset=UTF-8 
```

變成 :
```
# /usr/local/bin/curl --http2 -I --insecure https://ssl.demo.com
HTTP/2.0 200
server:nginx/1.12.0
date:Wed, 17 May 2017 08:45:06 GMT
content-type:text/html; charset=UTF-8
content-length:9
last-modified:Wed, 17 May 2017 07:32:24 GMT
etag:"2c0b72-9-54fb34a123200"
accept-ranges:bytes
```

